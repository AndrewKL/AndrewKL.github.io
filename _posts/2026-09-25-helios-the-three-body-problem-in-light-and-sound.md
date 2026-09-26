---
layout: post
title:  "Helios: the three-body problem in light and sound"
date:   2026-09-25
excerpt: "A gravity sandbox where every body glows, pulses and hums, and the closer two bodies get, the faster they pulse and wub. Why three bodies have no general solution, why they fly apart, and what happens when you turn a physics problem into something you can see and hear."
---

<ul class="actions special">
  <li><a href="/Helios3BodyProblemVisualizer/" class="button large">Launch Helios</a></li>
</ul>

[Helios](/Helios3BodyProblemVisualizer/) is a gravity sandbox that runs in the
browser. Click anywhere a few times to drop some bodies, press Play, and watch
them pull on each other. Every body is a glowing sphere that pulses and
hums a low sci-fi drone. The closer it gets to another body, the bigger and
brighter it gets and the faster it pulses and wubs. It never plays out the same way
twice, and that is the point: it is a toy built around one of the oldest
unsolved problems in physics.

## The three-body problem

Given three masses, their positions and their velocities, where will they be
at any later time? Newton answered this for *two* bodies in 1687. Two bodies
trace ellipses, parabolas or hyperbolas forever, and a formula tells you
exactly where they will be at any time you like. Add a third body and there is no such
formula.

## Why it's hard to solve

It is not for lack of trying. The problem has too many unknowns and too few
rules. Three bodies moving in space have 18 numbers that change over time:
three position and three velocity components each. Conservation of energy,
momentum and angular momentum pins down only 10 of them. In 1887 Heinrich
Bruns proved there are no other algebraic conserved quantities to make up the
difference.

Henri Poincaré went further around 1890 and showed that the motion generally
can't be reduced to a neat formula at all. Karl Sundman did find an infinite
series that technically solves it, in 1912, but it converges so slowly that
it is useless in practice.

So we simulate. Astronomers step the bodies forward in small time increments,
computing the forces and nudging every position and velocity a little at a
time, and so does Helios. That works, but every step adds a tiny error, and
with three bodies tiny errors don't stay tiny.

## Why it's unstable

Poincaré's other discovery was chaos. A difference in starting positions far
smaller than you could ever measure grows exponentially over time. Two nearly
identical setups soon look nothing alike, so long-term prediction is
impossible even though the laws themselves are exact. Nothing is random here;
it is deterministic and still unpredictable.

The mechanism is the close encounter. When two bodies pass near each other
they swap a lot of energy very quickly, and one can be flung onto a wide
orbit or out of the system entirely. That is the usual ending: most
three-body systems eventually throw one body out, and the other two settle
into a tight pair carrying away the leftover energy.

There are rare exceptions, special starting positions that repeat forever.
Euler found three bodies in a line in 1767, and Lagrange found them at the
corners of an equilateral triangle in 1772; Jupiter's Trojan asteroids ride
that triangle. In 1993 Cris Moore found three equal masses chasing each other
around a figure-eight. But most of these orbits are balanced on a knife-edge.
Nudge them and chaos takes over.

## Science, art and music

Helios is honest physics. Every body pulls on every other body with Newton's
inverse-square law, and a velocity Verlet integrator steps it forward 240
times a second, with a fixed timestep so the result doesn't depend on your
frame rate. There are two deliberate compromises, and the info panel in the
app says so. Gravity is softened at very short range so bodies pass through
each other instead of reaching infinite speeds. And a weak attractor sits at
the center of the screen, pulling strays back instead of letting the usual
ejection end the show.

The art is in what the physics is made to look like. The quantity that
matters most in a three-body system is invisible: how close each body is to
its nearest neighbor, because that is where the close encounters, the energy
swaps and the chaos come from. Helios draws that. As two bodies approach,
each grows, its core burns whiter, its pulse speeds up and deepens, and it
stretches toward its neighbor like a body being pulled apart by tides. A near
miss reads as a flare. The drama on screen is the same thing that is making
the math hard.

The music does the same thing for your ears. Each body is its own voice: two
detuned sawtooth waves and a sub-octave sine running through a resonant
filter, with a slow oscillator sweeping the filter open and closed. That
sweep is the "wub." Far from everything, a body wubs lazily, about one and a
half times a second. As it closes in on another body, the wub speeds up to
ten times a second and the tone gets brighter and louder. Each body is panned
left or right by its position on screen, so you can hear where it is.

The notes are chosen so the chaos stays listenable. Each new body takes the
next note of a pentatonic scale, the five-note scale with no half steps in
it, so however many bodies you add and however they move, the
chord never clashes. The physics decides the rhythm and the scale keeps the
harmony.

Put together, a simulation becomes a composition that no one wrote. You
choose the starting positions, and after that Newton writes the score: long
quiet drifts where every voice wubs slowly and apart, then a close pass where
two voices race and flare together, and one body gets slung across the stereo
field. Place the same bodies a few pixels differently and you get an entirely
different piece. That is chaos, heard rather than read about.

## Try it

[Helios is live here](/Helios3BodyProblemVisualizer/), and the code is on
GitHub:
[AndrewKL/Helios3BodyProblemVisualizer](https://github.com/AndrewKL/Helios3BodyProblemVisualizer).
It is TypeScript and three.js, with the Web Audio API for the sound and no
other libraries. Turn your sound on, click a handful of times, and press Play.

<ul class="actions special">
  <li><a href="/Helios3BodyProblemVisualizer/" class="button large">Launch Helios</a></li>
</ul>
