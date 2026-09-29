# Aether Isle

A small procedural island at the edge of the sky, explored in first person. The whole game is one self-contained file, [`index.html`](index.html): no build step, no assets, no server. Everything except three.js (loaded from a CDN) is generated at runtime: terrain, foliage, sky, water, creatures and audio.

**Play it live: https://az9713.github.io/sonnet-5.5-3d-world-demo/**

Best in desktop Chrome with hardware acceleration on. It needs an internet connection on first load to fetch three.js. You can also just double-click `index.html`.

## Goal

Three glowing light-stones are scattered across the island (lake shore, forest, a far hill). Carry each one to the pedestal of the same colour in the ruin. When all three are placed the ancient spire awakens: its runes ignite in sequence and a beam of light fires into the sky.

## Controls

| Key | Action |
| --- | --- |
| Click | Begin / resume (locks the mouse) |
| W A S D | Move |
| Mouse | Look |
| Shift | Sprint |
| Space | Jump |
| E | Pick up / drop an object |
| Left click | Throw the carried object |
| T | Cycle weather (clear, rain, fog) |
| M | Mute |
| Esc | Pause |

## Features

- **Terrain:** noise-generated hills, beach, cliffs and a sheer island edge that falls into a cloud sea. Three zones: forest, ancient ruin, lake and wetland.
- **Vegetation:** about 42k wind-swaying grass blades placed by the vertex shader around the player (they bend as you walk through them), flowers, reeds, shrubs and trees.
- **Day/night:** a 180 s cycle with sunrise, sunset, moon, stars and a faint aurora. Sky, fog, lighting and water all follow the time of day.
- **Weather:** clear, rain and fog blend smoothly; rain particles are one draw call, with synthesized rain and thunder.
- **Water:** shader-faked refraction, Fresnel, caustics and shore foam, plus a planar reflection (half resolution, at most 30 Hz, skipped when off-screen). Ripples follow you and the rain.
- **Creatures:** flocking birds (cohesion, separation, alignment) that leave at dusk, hares that wander, notice you and flee, and glowing fireflies at night.
- **Physics:** pick up, drop and throw objects with gravity, bounce, damping and rolling. Anything that falls off the island or sinks into deep water respawns at its origin with a light pulse.
- **Audio:** Web Audio only. Wind, birds, crickets, rain, thunder, footsteps that differ on grass, sand, stone and water, and puzzle sounds.
- **Adaptive quality:** watches frame time and steps quality down or up (reflection resolution and rate, then shadow map size, then render scale, then an analytic reflection).

## Performance notes

Roughly 48 draw calls and about 414k triangles per frame (including the shadow and reflection passes). One shadow-casting sun, two point lights, no full-screen post-processing. Not yet profiled on real hardware; the target is 60 FPS on an RTX 3050 laptop GPU.

Add `?debug` to the URL to show the GPU name, draw calls, triangle count, frame time, render scale and quality tier.

## Hosting

The site is published by the workflow in `.github/workflows/pages.yml`. If the first run cannot enable Pages itself, turn it on once under **Settings → Pages → Source: GitHub Actions**, then re-run the workflow.

## Known limits

Colliders are circles and objects are spheres (no object-to-object collision), there are no rain splash particles, and flowers and reeds do not receive shadows.
