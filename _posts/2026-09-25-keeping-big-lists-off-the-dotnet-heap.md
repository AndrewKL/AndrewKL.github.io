---
layout: post
title:  "Keeping big lists off the .NET heap"
date:   2026-09-25
excerpt: "Millions of small objects in a List<T> are cheap to read and expensive to keep around, because the garbage collector has to trace every one of them. Serializing them into native memory instead cuts memory use by almost two-thirds and makes a full GC almost free, but every read gets slower. A small C# library, a benchmark and the trade-offs."
image: "/images/off-heap/benchmark.jpg"
image_alt: "Four bar charts comparing 5 million records stored three ways: a List<T> on the heap, off heap with a boxed serializer, and off heap with a typed serializer. Build time is 1,503 ms on heap versus 546 and 476 ms off heap. Enumerating once takes 84 ms on heap versus 749 and 392 ms off heap. A full garbage collection while the data is alive takes 123.6 ms on heap versus 0.1 and 0.2 ms off heap. Memory held is 560 MB on heap versus 204 MB for both off-heap versions."
---

<ul class="actions special">
  <li><a href="https://github.com/AndrewKL/CSharpOffHeapStorage" class="button large">View the code on GitHub</a></li>
</ul>

When I was a new SDE at ZS Associates, more than 15 years ago, I spent a
significant amount of time playing with very large in-memory lists. Calling
`ToList()` on a few million records is easy to write, and it's easy to miss
what it costs: memory climbs, the garbage collector kicks in over and over,
and more and more of the program's time goes to the runtime cleaning up
after it.

A few years later, in 2014, I wrote a small library to test an idea: what if
the list didn't live on the managed heap at all? Serialize every object into
one big block of memory, and turn the bytes back into objects only while you
are looping over them. I recently got it building again on .NET 10, moved its
storage into native memory, and wrote a benchmark to see whether the idea
holds up.

## Why a big list hurts

.NET's garbage collector is generational. New objects start in generation 0,
which is collected often and cheaply, on the bet that most objects die young.
Objects that survive a few collections get promoted to generation 2, which is
only collected occasionally. To collect, the GC has to find every object that
is still reachable, which means following every reference from every live
object. The cost of that tracing grows with the number of live objects, not
with how much memory they take up.

A big list breaks the generational bet. Every record you add survives, because
the list holds on to it. So each gen 0 collection finds almost nothing to free
and has to copy the survivors up a generation, and the gen 1 and gen 2
collections that follow have to trace millions of objects and references each
time. Five million records with one string each is ten million objects, plus
the list's internal array, which is big enough to go on the Large Object Heap.
Every one of those objects also carries a 16-byte header and is reached
through an 8-byte pointer.

None of this shows up as an error. The program just gets slower, and it gets
slower the most in the code that has nothing to do with the list, because a
long gen 2 collection pauses every thread.

## Getting the data off the heap

The library is `OffHeapIEnumerable<T>`. You give it any `IEnumerable<T>`, and
it walks through it once, writing each object's properties into a stream with
a `BinaryWriter`: an `int` is 4 bytes, a `string` is a length prefix plus
UTF-8, and so on. Enumerating it reads the stream back and builds a fresh
object for each record as you reach it. To the GC, the whole data set is
nothing but the stream.

What that stream sits on has changed three times. The first version used a
plain `MemoryStream`, which keeps everything in one `byte[]`. That fails on
big data sets, because a single array needs one contiguous run of memory and
a `MemoryStream` can't grow past 2 GB. The 2014 version swapped in a
`MemoryTributary`, which is a stream backed by a list of 64 KB arrays. The
chunks are small enough to stay off the Large Object Heap, and the GC only
sees a few thousand of them instead of millions of objects. They are still
managed memory, though.

The version I have now allocates its 1 MB blocks with `NativeMemory.Alloc`,
outside the GC heap entirely. The GC never scans them, never moves them and
doesn't count them. The catch is that nothing frees them for you. The
collection is `IDisposable`, and you have to dispose it, the same as a file
handle.

## The benchmark

The benchmark builds 5 million records, each with an `int`, a `string`, a
`decimal`, a `double` and a `bool`, and stores them three ways. The first is a
plain `List<T>`. The second is the off-heap collection with the original
serializer. The third is the off-heap collection with a newer, faster
serializer I'll come back to. For each one it measures how long it takes to
build, how long it takes to loop over once, how much memory it holds, and how
long a forced full, blocking garbage collection takes while the data is still
alive. That last number is the one this whole project is about.

| | `List<T>` | Off heap, boxed | Off heap, typed |
|---|---:|---:|---:|
| Build | 1,503 ms | 546 ms | 476 ms |
| Time in GC pauses while building | 916 ms | 29 ms | 23 ms |
| Gen 2 collections while building | 6 | 0 | 0 |
| Memory held | 560 MB | 204 MB | 204 MB |
| Full GC while the data is alive | 124 ms | 0.1 ms | 0.2 ms |
| Enumerate once | 84 ms | 749 ms | 392 ms |

The off-heap version is three times faster to build, and the reason is in the
second row. More than half of the `List<T>` build time is the garbage
collector copying and tracing records that were never going to die. The
off-heap version makes plenty of garbage while building too, but all of it
dies young, which is the case the GC is designed for.

It also takes 2.7 times less memory. The list pays a 16-byte header for every
record and every string, an 8-byte pointer to each, and two bytes per character
in UTF-16. The stream pays a byte or two of framing instead.

A full collection with the list alive takes 124 ms, and that grows with the
list. With the data off heap it takes a tenth of a millisecond, because
there's nothing left to trace.

And then there's the last row. Looping over the list takes 84 ms, because the
objects are already there. Looping over the off-heap version is about 5 to 9
times slower, because every pass has to build 5 million new objects and 5 million
new strings from bytes.

## Boxing, and a faster serializer

The original serializer made that last row worse than it needed to be. It
worked through reflection in the most general way: a list of properties, each
with a getter and setter delegate typed as `object`, and a `switch` on the
property's type. Every `int`, `bool`, `decimal` and `double` it read got boxed
into a short-lived heap object on the way to its setter. That's four extra
allocations and five delegate calls per record.

The typed serializer builds one reader and one writer per type with
expression trees and compiles them, so the result is the same code you would
write by hand:

```csharp
reader => new Record
{
    Id = reader.ReadInt32(),
    Name = reader.ReadBoolean() ? reader.ReadString() : null,
    Price = reader.ReadDecimal(),
    Score = reader.ReadDouble(),
    Active = reader.ReadBoolean()
}
```

That halves the garbage made while enumerating, down to exactly one record
and one string each, and it makes enumeration 1.3 to 1.9 times faster across
runs. It is still slower than walking a list. Most of the remaining cost is
going through `BinaryReader` and `Stream` for every 4- or 8-byte field; reading
straight out of the native blocks would be the next step.

## On heap vs off heap

**On heap** is the right default, and it isn't close for most programs.

- Reading is as fast as it gets. The objects already exist, so a loop is just
  following pointers.
- You get random access, updates in place, LINQ, and any type you like,
  including ones with nested objects and collections.
- The GC frees it all for you, and there is no serialization format to keep in
  sync with your classes.
- The cost is the GC. Every live object is something to trace, so collections
  get slower as the data grows, and it's the whole process that pauses, not
  just the code using the list.
- Small objects are memory-hungry: 16 bytes of header each, 8 bytes per
  reference, and UTF-16 strings.

**Off heap** trades read speed for everything else.

- A full GC stays at a tenth of a millisecond no matter how much data you
  hold, and building the data set doesn't push the rest of the program through
  gen 2 collections.
- It takes much less memory, because there are no object headers or
  pointers, and strings are stored as UTF-8.
- Every read deserializes. If you loop over the data many times, you pay that
  cost every time, and it is several times the cost of reading a list.
- It only goes forward. There's no indexing and no updating in place; you
  append and you enumerate.
- It only handles the types the serializer knows about. Here that's `int`,
  `bool`, `decimal`, `float`, `double` and `string` properties on a class with
  a public parameterless constructor.
- You own the memory. Forget to dispose it and the GC won't know it's there,
  so the finalizer that would eventually free it may not run for a long time.
- The objects you get back are copies. Changing one doesn't change what's
  stored.

The trade is worth it when the data set is big, lives a long time, is read
only a few times, and shares a process with code that cares about pause
times, like a web server that keeps a large cache or reference data set in
memory. It isn't worth it for a list you build, loop over once and throw away.
A few million short-lived objects are exactly what the GC is good at.

## What I'd reach for today

There are other ways to get most of the way there. Background GC does most
of the gen 2 work while your threads keep running, although the gen 0 and
gen 1 collections that promote a growing list still stop them. If your records can be `struct`s with no reference fields, an array of
them is one object to the GC, and the GC doesn't look inside it, which gets
you most of the benefit with none of the serialization. And if the data is
really a table, an in-process columnar engine like DuckDB, or Apache Arrow,
will do this better than a hand-rolled stream.

Still, the core idea holds. The garbage collector's cost is in the number of
live objects, not their size, so when you have millions of them, the best
thing you can do is give it fewer to look at.

The code, the tests and the benchmark are on GitHub at
[AndrewKL/CSharpOffHeapStorage](https://github.com/AndrewKL/CSharpOffHeapStorage).
Running
`dotnet run -c Release --project OffHeapStorage.Benchmark -- 1000000 5000000`
reproduces the numbers on your own machine.

<ul class="actions special">
  <li><a href="https://github.com/AndrewKL/CSharpOffHeapStorage" class="button large">View the code on GitHub</a></li>
</ul>
