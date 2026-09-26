---
layout: post
title:  "Painting water with the Navier–Stokes equations"
date:   2026-09-25
excerpt: "A 2D water simulation that pours over mossy rocks, painted in brush strokes that run from deep blue to white foam the faster the water moves, with a rush, splashes and a swelling surf you can hear. What the Navier–Stokes equations say, why they are still unsolved, and how a few thousand particles fake a waterfall."
image: "/images/navier-stokes/painted-waterfall.jpg"
image_alt: "The water simulator mid-flow, painted like an impasto oil painting: water drawn as brush strokes runs deep blue in still pools and turns white where it pours fast over slate boulders topped with yellow-green moss, against a dark, mossy backdrop. A legend in the corner reads slow to fast, 0 to 2.6 m/s, and the Pause, Reset, Flow, Surge, Info and Mute controls sit at the bottom."
---

<ul class="actions special">
  <li><a href="/NavierStokesSimulation/" class="button large">Launch the water simulator</a></li>
  <li><a href="https://github.com/AndrewKL/NavierStokesSimulation" class="button large">View the code on GitHub</a></li>
</ul>

The [water simulator](/NavierStokesSimulation/) is a small river that runs in
the browser. Click once and water pours from a ledge high on the left, crashes
onto a row of mossy boulders, pools behind them, spills over and runs out the
right side. Drag anywhere to drop in a new boulder and watch the water find
its way around it. The whole scene is painted in brush strokes: still water is
deep blue, and the faster the water moves, the paler it gets, until the
fastest water on screen is white foam. It sounds like water too, with a rush
that follows the current, splashes where it lands, and a low surf that swells
as it sloshes.

## The Navier–Stokes equations

Water, air, honey and blood all move by the same two rules. Claude-Louis Navier
wrote them down in 1822 and George Gabriel Stokes put them on firm footing in
1845. They are Newton's *force equals mass times acceleration*, applied to
every tiny parcel of a fluid.

The first rule is about momentum. A parcel of water speeds up or turns because
pressure pushes on it, gravity pulls on it, and viscosity, the fluid's internal
friction, drags it toward the speed of its neighbors. It also carries its own
velocity along as it moves, and that self-carrying term is what makes flow
curl back on itself into swirls and eddies.

The second rule is that water can't be squeezed. Whatever flows into any small
region has to flow out again. Pressure is exactly the force needed to keep
that true: it builds up wherever water is being crammed together and pushes
it apart.

## Why they're still unsolved

Two short equations, and still nobody knows whether they always make sense. In
three dimensions, no one has proved that smooth solutions always exist, or
that the velocity can't suddenly blow up to infinity at a point. It is one of
the Clay Mathematics Institute's seven Millennium Prize Problems, with a
million-dollar reward for either answer.

The practical problem is turbulence. How chaotic a flow gets depends on its
Reynolds number: speed times size, divided by viscosity. A stream running at a
meter per second over a meter-sized rock has a Reynolds number around a
million. At that point the flow breaks into eddies, the eddies break into
smaller eddies, and the cascade doesn't stop until it reaches swirls tens of
microns across, where viscosity finally turns the motion into heat. Resolving
all of that in 3D would take something like ten trillion grid cells. Engineers
don't do it. They model the small eddies statistically and simulate only the
big ones, and even that takes supercomputers.

## How a few thousand particles fake a waterfall

This simulation cheats in the way film effects cheat. It uses a method called
FLIP, short for Fluid-Implicit-Particle. Brackbill and Ruppel invented it for
plasma physics in 1986, and Zhu and Bridson brought it to computer graphics in
2005. The water is about six thousand particles, each carrying a position and
a velocity. Every step, their velocities are copied onto a coarse 120 × 60
grid. A pressure solve adjusts the grid until no cell gains or loses water,
and the change is copied back to the particles. The particles give you a free
surface, where water meets air, for free, and they carry momentum without
smearing it, which is what keeps the water lively and splashy instead of
syrupy.

It looks right because at this scale water is dominated by gravity, inertia
and incompressibility, and FLIP gets those three right. Jets fall in
parabolas, pools find their level, waves travel and water piles up against
rocks. What it gets wrong is everything small. There is no viscosity term, the
only damping comes from numerical side effects, and the grid cells are about a
thousand times too coarse to see real turbulence. It is also 2D, and 2D
turbulence is a different animal: small swirls merge into big, long-lived
whirlpools instead of shattering into smaller ones. The big lazy eddies you
see in the pools are partly that.

## Science, art and sound

The physics decides where every bit of water goes. The art is in what it is
made to look like, and the look comes from an impasto painting of a woodland
waterfall: white water tumbling between dark slate boulders with bright moss
on top.

Color is speed. The quantity your eye can't normally see in moving water is
how fast each part of it is going, so the simulation paints it. Each particle's
speed is compared with the fastest water currently in the scene. The slowest
water is deep blue, faster water is paler, and the fastest is white. The scale
follows the water, so a gentle trickle and a torrent both use the full range.
Press Surge and the whole jet flashes white, then the colors settle back over
a second or two as the scale catches up. This happens to be how rivers look.
Real white water is white because it is fast and broken up, and still pools
are dark.

Every particle is a brush stroke. Each one is stretched along the direction
it is moving, with bristle streaks and a little variation in tone, so fast
water becomes long, thin, pale streaks that trace the flow, and still water
becomes short horizontal dabs, like a painter's pool. The rocks are painted
the same way: slate strokes with light catching the top, a band of moss that
creeps down the sides, and a seed per rock so it looks the same every frame.

The sound is built the same way, from the physics up, with no recorded audio
at all. A rush of filtered noise follows the average speed of the water,
getting louder and brighter as it speeds up, and pans toward wherever the fast
water is. Splashes are short bursts of noise, triggered when water hits
something hard enough to change its velocity sharply. Under both is a low surf
that swells and fades like waves on a shore and grows as the water sloshes up
and down. Draw a boulder into a quiet pool and you hear the slap. Press Surge
and the whole river roars.

## Try it

[The water simulator is live here](/NavierStokesSimulation/), and the code is
on GitHub:
[AndrewKL/NavierStokesSimulation](https://github.com/AndrewKL/NavierStokesSimulation).
It is TypeScript and WebGL2, with the Web Audio API for the sound and no other
libraries. Turn your sound on, click to let the water flow, and drag a few
boulders into its path.

<ul class="actions special">
  <li><a href="/NavierStokesSimulation/" class="button large">Launch the water simulator</a></li>
  <li><a href="https://github.com/AndrewKL/NavierStokesSimulation" class="button large">View the code on GitHub</a></li>
</ul>
