# The Jupiter Shield ☄♃

**Live demo: https://nikolai3d.github.io/jupiter-defense/**

An interactive WebGL experiment answering the plea:

> *"People of science, hear my plea: is it true that Jupiter quietly takes the killer comets for us? Run both universes side by side, and count."*

Two copies of the same young solar system run simultaneously, seeded with the **same** swarm of comets on Earth-threatening orbits — identical positions, velocities, and random seed. The left system keeps its giant planet at 5.2 AU; the right one never formed it. Nothing is scripted: each comet is integrated through the restricted three-body problem in real time, and a live ledger counts Earth impacts, close calls, ejections, comets swallowed by Jupiter, and surviving Earth-crossers in both worlds. Every divergence between the two ledgers is Jupiter's doing.

## Running it

No build step, no dependencies — a single self-contained HTML file using raw WebGL 2.

**Option 1: just open the file**

```sh
open index.html          # macOS
xdg-open index.html      # Linux
start index.html         # Windows
```

**Option 2: serve it locally**

```sh
python3 -m http.server 8000
# then visit http://localhost:8000
```

Requires a browser with WebGL2 (any modern Chrome, Firefox, Safari, or Edge).

## Using the simulator

- **Drag** to orbit the camera, **scroll / pinch** to zoom — one camera, two worlds.
- **Comets per world** — how many test comets each universe gets (the CPU engine tires past ~10,000; the GPU engine keeps going into the hundreds of thousands).
- **Giant planet mass** — from a puny 0.03 M♃ (barely helps) to a 6 M♃ super-Jupiter (clears the sky). Watch the ejection counter scale.
- **Comet family** presets:
  - *Jupiter-family swarm* — comets on Jupiter-crossing orbits with perihelia in the inner system; the population Jupiter polices hardest.
  - *Long-haul visitors* — Halley-type orbits from far out; eviction is slower but just as final.
  - *Inner rogue belt* — strays Jupiter can't reach. No encounters, no eviction: the shield largely switches off. (The honesty corner.)
- **Compute engine**: CPU · float64 or GPU · float32 — the same adaptive velocity-Verlet in two arithmetics. Switching hands the live state across, no re-deal.
- **Speed** up to 500 simulated years per second. If your hardware can't keep up, comets carry a private time debt and catch up later — speed degrades, physics never does.
- **Charts** track Earth-crossing threats remaining and cumulative impacts in both worlds; the **verdict panel** calls the score, including the times Jupiter is *losing* (it happens — see below).
- **Integrator check**: median |Δa/a| in the planet-free world, where every comet's semi-major axis should be exactly conserved. Any drift shown is the integrator's sin, not physics.

## The science in one paragraph

A comet that crosses Jupiter's orbit will sooner or later slingshot past it, and each pass random-walks the comet's orbital energy toward one of two absorbing ends: ejection to interstellar space, or decoupling from Earth's orbit (a few hit the planet itself, as Shoemaker–Levy 9 did in 1994). So an Earth-crossing comet in the Jupiter world has a short life expectancy, while its twin in the other world re-rolls its impact dice every perihelion, forever. That's the famous "Jupiter shield" (Wetherill 1994) — but honesty requires the counterpoint: Horner & Jones (2008–2012) showed Jupiter is a Jekyll-and-Hyde neighbour that also nudges main-belt asteroids and Oort-cloud visitors *inward*, and an intermediate-mass giant can be worse than nothing. This simulation shows the population where the shield story is strongest, and hands you the sliders to find where it isn't.

## Implementation notes

- Restricted three-body dynamics: each comet feels the Sun and (in the left world) Jupiter, which ride an exact analytic two-body solution — the Sun visibly orbits the barycenter, and planetary motion carries zero integration error.
- Adaptive velocity-Verlet with a per-comet timestep, η = 0.035, clamped near Earth so encounters are resolved below the capture radius — no tunnelling through the target.
- Because test particles never interact, the problem is O(N), and the GPU engine advances every comet hundreds of adaptive substeps per frame inside a single fragment shader pass (state lives in RGBA32F textures, triple MRT: position+status, velocity, time-debt+encounter-flags). The CPU engine is the identical scheme in float64.
- Earth's capture radius is inflated to 0.004 AU (~75× its gravitationally-focused radius) so impact statistics arrive in minutes rather than years; the inflation is identical in both worlds, so the impact *ratio* — the number this page exists to measure — stays meaningful. Same for Jupiter (0.01 AU) and the Sun (0.02 AU).
- Units: G = 1, M☉ = 1, 1 length unit = 1 AU, one year = 2π time units, 1 M♃ = 9.54×10⁻⁴ M☉.
- No external requests of any kind — everything is inline.

## Siblings

Part of a family of honest single-file WebGL experiments: [galaxy_collide](https://github.com/nikolai3d/galaxy_collide) (O(N²) galactic collisions) and [double_world](https://github.com/nikolai3d/double_world) (a habitable binary-planet system).
