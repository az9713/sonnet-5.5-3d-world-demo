# Browser 3D World — Capability Stress Test

Build the most impressive **interactive 3D world you can create in a single self-contained HTML file**.

This is a capability test. I care about **engineering quality, visual coherence, interaction depth, procedural generation, performance, and polish**—not merely satisfying a checklist.

## DELIVERABLE

Produce exactly:

`index.html`

It must run by **double-clicking the file in Chrome**. No server, npm, build system, bundler, installation step, or additional files.

### Allowed
- Three.js loaded from a public CDN (see "Three.js loading" below).
- Vanilla HTML, CSS, JavaScript, WebGL/GLSL, Canvas APIs, and Web Audio.
- System/generic CSS font stacks for all UI text (for example `font-family: system-ui, "Segoe UI", Roboto, sans-serif`).

### Forbidden
- External images
- External textures
- External 3D models
- External audio
- External fonts (no `@font-face`, no font files, no Google Fonts links)
- External JSON/data assets
- Additional local files

Everything other than Three.js must be generated procedurally inside `index.html`.

### Three.js loading (this is the most likely failure point — follow exactly)

- Pin an exact version: **three@0.160.0**. Never use unversioned or `@latest` URLs.
- Load it as an ES module from an **inline** `<script type="module">`. The classic `build/three.js` and `build/three.min.js` builds were removed in r160, so do not use `<script src=".../three.min.js">`.
- Do not reference any local script or module files (they are blocked on `file://`). All of your code is inline.
- Do not use `examples/jsm` addons (they import the bare specifier `"three"` and would require an import map). Implement what you need yourself.
- Primary URL: `https://unpkg.com/three@0.160.0/build/three.module.js`
  Fallback URL: `https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js`
- Load with dynamic `import()` inside `try/catch` (a static `import` cannot be caught): try the primary, then the fallback.
- If both fail, show a readable full-page message ("Three.js could not be loaded from the CDN. This file needs an internet connection on first load.") instead of a blank page.
- Use only Three.js APIs that exist in r160, with the r160 signatures.

### Size discipline (a truncated file is a failed deliverable)

- The file must be complete and end with `</html>`. Budget for roughly 2,500 lines or fewer. Keep comments to section headers and non-obvious rationale; prefer compact helpers over repetitive code.
- If you judge you cannot fit everything within your output limit, cut scope from the bottom of the PRIORITY ORDER upward. Never truncate the file and never leave stubs. State any cuts in the final note.

---

# PERFORMANCE TARGET

Target hardware:

- Windows laptop
- Chrome
- NVIDIA GeForce RTX 3050 Laptop GPU

The experience should target **60 FPS during normal exploration**.

Include a small unobtrusive FPS counter.

Design intelligently for performance. Prefer convincing procedural techniques over brute-force geometry or particle counts.

Avoid unnecessary:
- draw calls
- allocations inside animation loops
- shadow-casting objects
- transparent overdraw
- high-poly geometry
- expensive full-screen effects

Use instancing, object pooling, LOD/culling, simplified physics, and shader-based effects where appropriate.

## Performance budget (hard limits — these override any other requirement they conflict with)

- **Draw calls:** at most about 150 per frame during normal exploration (`renderer.info.render.calls`). **Triangles:** at most about 500k (`renderer.info.render.triangles`).
- **Pixel ratio:** initial `renderer.setPixelRatio(Math.min(devicePixelRatio, 1.5))`.
- **Lights:** one directional light (the only shadow caster), one ambient/hemisphere light, and at most 2 dynamic point lights in the entire scene.
- **Shadows:** a single 2048×2048 shadow map on the directional light, with its frustum fitted to roughly 60–80 m around the player and snapped to the texel grid to avoid shimmering. Only large structures may cast shadows: ruin, trees, physics objects. Grass, flowers, shrubs, reeds, creatures, particles, and water do not cast shadows.
- **Water reflection:** at most one extra scene render for the water surface, at no more than half the main render resolution and at most 30 Hz, containing only the sky, distant terrain, and landmarks. Skip it when the water is outside the view frustum. At the lowest quality tier, replace it with an analytic sky-colour reflection in the shader.
- **Water refraction:** must be faked in the water shader (depth-based colour, normal-distorted lookup into a cheap procedural bed/caustic term). Never re-render the scene for refraction.
- **Transparency:** only these surfaces may be transparent: water, rain (one draw call), the cloud layer (a small number of large meshes — no stacks of hundreds of sprites), and additive glow sprites (one `Points` draw call per glow type). Set `depthWrite = false` on them and keep their geometry bounds tight. Fog must be scene fog or shader fog, not screen-covering overlay planes.
- **Post-processing:** no full-screen post-processing passes. The fall/respawn fade to white is a DOM overlay.
- **Adaptive quality controller (required):** track smoothed frame time (about a 1 s window). If it exceeds about 20 ms for 2 s, step down one quality tier; if it stays below about 13 ms for 5 s, step up one tier. Change at most one tier per 2 s and add hysteresis so it never oscillates. Tier order when stepping down: (1) reduce reflection resolution and rate, (2) shadow map 2048 → 1024, (3) lower render scale in 0.1 steps down to a floor of 0.6, (4) at the lowest tier, replace the reflection pass with the analytic reflection. Stepping up reverses the order.

---

# WORLD

Create a small but dense **floating island at the edge of the sky**, explored in first person.

The world should feel like a coherent place rather than a collection of demos.

The player should immediately be able to see at least one intriguing distant landmark encouraging exploration.

---

## 1. PROCEDURAL TERRAIN

Generate the island procedurally.

The terrain should contain:

- rolling hills
- grassland
- a sandy beach
- steep cliffs
- an edge that visibly falls away into clouds/the sky
- terrain variation derived from procedural noise rather than a simple fixed mesh

Include at least **three visually and mechanically distinct zones**, such as:

1. Forest
2. Ancient ruin
3. Lake / wetland

Transitions between zones should feel natural.

Add substantial procedural vegetation.

Grass, flowers, shrubs, trees, reeds, or similar foliage should respond visibly to wind.

Use instancing or another efficient technique where appropriate.

---

## 2. DAY / NIGHT CYCLE

Implement a continuous full day/night cycle lasting approximately:

**180 seconds**

Include:

- moving sun
- moving moon
- sunrise
- daylight
- sunset
- night
- stars appearing after dark

Time of day should continuously affect:

- sky color
- fog color/density
- directional lighting
- ambient lighting
- shadow appearance
- water appearance
- creature behavior where appropriate

Avoid abrupt state changes.

The cycle should create noticeably different moods at dawn, midday, sunset, and night.

---

## 3. DYNAMIC WEATHER

Weather should transition naturally between:

- Clear
- Rain
- Fog

Weather changes automatically over time.

Press:

**T**

to force progression to the next weather state.

### Rain should include

- procedural rain particles
- darker / cooler environmental lighting
- reduced visibility
- rain audio synthesized using Web Audio
- visible impact on the mood of the world

### Fog should include

- substantially reduced visibility
- atmospheric depth
- lighting changes appropriate to dense mist

Weather transitions should interpolate smoothly rather than switch instantly.

---

## 4. WATER

Create a lake or pond substantial enough to enter.

The water must be visibly animated.

Implement at least one convincing approximation of:

- reflection
- refraction
- Fresnel reflection
- depth-dependent color
- procedural distortion

Use shaders if useful.

When the player walks through shallow water, generate visible **ripples around the player**.

Water should also affect footsteps/audio.

---

## 5. LIVING WORLD

Include at least **two distinct creature systems with autonomous behavior**.

### Flying creatures

For example:

- birds
- magical flying creatures

Implement simple flocking behavior inspired by boids:

- cohesion
- separation
- alignment

They should move through the world rather than simply orbit one fixed point.

### Ground creatures

Include animals or creatures that:

- wander
- change direction
- avoid obstacles approximately
- detect the player
- flee when approached

### Fireflies

At night:

- fireflies appear
- move organically
- emit visible light/glow
- disappear or become inactive during daylight

Firefly glow must be faked with additive point sprites (a single draw call, shader-driven flicker). Never create one real light per firefly; the scene-wide limit of 2 dynamic point lights in the performance budget applies.

The island should feel noticeably more alive because of these systems.

---

# 6. PHYSICAL INTERACTION

The player must be able to interact with physical objects.

Controls:

**E** — pick up / release an object (release = gently drop it in front of the player)

**Left mouse button** — while carrying an object, throw it along the view direction (a fixed impulse plus a portion of the player's current velocity)

While carrying an object, allow the player to throw it.

Implement lightweight physics including:

- gravity
- velocity
- terrain collision
- bouncing
- damping

The simulation does not need a full physics engine.

---

# 7. ENVIRONMENTAL PUZZLE

Place a puzzle in the ruin.

Example structure:

- Three glowing stones exist elsewhere in the world.
- The player must find them.
- Each stone can be physically carried.
- The ruin contains three matching pedestals.
- Stones must be placed correctly.

The puzzle must use the physical interaction system rather than simply clicking three switches.

When solved, create a **visually satisfying world event**, such as:

- an ancient portal activating
- ruins rising from the ground
- a beam of light firing into the sky
- floating stones assembling
- a hidden structure opening

The result should be visible from a distance and feel rewarding.

Add sound and lighting changes to reinforce the event.

### Puzzle rules (unambiguous)

- There are three stones in three distinct hues (for example cyan, amber, magenta). Each pedestal has a glyph ring glowing in exactly one of those hues.
- A stone counts as placed only when it comes to rest within a small radius of the pedestal whose hue matches it. It then snaps onto the pedestal, becomes non-carriable, and the pedestal lights up. Placement order does not matter.
- A stone that comes to rest on a non-matching pedestal is not accepted: it is gently ejected with a dim flash and a discordant tone, nothing is consumed, and it can be picked up again.
- The puzzle is solved when all three stones are placed, which triggers the world event.
- **No softlocks.** Every stone must always be recoverable. If a stone falls below the island's kill height, leaves the playable bounds, or rests in water too deep to wade for more than about 3 seconds, it returns to its original spawn point with a visible light pulse and a soft sound. If the player is carrying a stone when they fall, the stone is dropped and follows the same rule. Ordinary (non-puzzle) throwable objects that fall off the island or into deep water respawn at their origin points the same way.

---

# 8. PROCEDURAL AUDIO

Use **Web Audio only**.

No audio files.

Create a lightweight generative soundscape containing:

- wind
- daytime bird ambience
- nighttime crickets
- rain
- footsteps

Footsteps should differ depending on surface:

- grass
- sand
- shallow water

Add occasional spatial or environmental sounds if performance permits.

Include a visible mute toggle.

Audio must begin only after user interaction so browser autoplay restrictions are respected.

---

# 9. FIRST-PERSON CONTROLS

Implement:

- WASD — movement
- Mouse — look
- Shift — sprint
- Space — jump
- E — interact / carry / release
- Left mouse button — throw carried object
- T — cycle weather
- Esc — release pointer lock (this pauses the game; see below)

Use pointer lock.

Movement should have subtle acceleration/deceleration rather than feeling completely binary.

Implement gravity and terrain following.

### Pause behavior

- Whenever pointer lock is lost for any reason (Esc, alt-tab, or a failed lock request), the game pauses. The day/night clock, weather timers, creature and physics simulation, and ambient audio (suspend the `AudioContext`) all freeze, and a minimal "PAUSED — CLICK TO RESUME" overlay, styled like the start screen, is shown.
- Clicking re-requests pointer lock and resumes. Reset the frame timer on resume so the paused interval is never simulated.
- Pointer-lock requests can be refused (for example shortly after Esc). Handle both the promise rejection and the `pointerlockerror` event without throwing, and keep the overlay up so the next click retries.
- Ignore all gameplay keys (T, E, movement, mouse throw) while paused.

---

# 10. START SCREEN

Before entering the world, show a minimal cinematic title/start screen.

Explain the controls briefly and prominently show:

**CLICK TO BEGIN**

Pointer lock and audio initialization should occur after this click.

---

# 11. HUD

Keep the HUD minimal.

Show:

- current time of day
- current weather
- FPS
- mute state
- contextual interaction hint when appropriate

Do not clutter the screen.

---

# 12. FALL / RESPAWN

If the player falls from the floating island:

1. detect the fall
2. fade the screen toward white
3. respawn the player at a safe location
4. smoothly fade back into the world

Do not abruptly teleport without visual treatment.

Any carried object is released the moment the fall is detected and is then subject to the recovery rules in section 7.

---

# 13. VISUAL POLISH

Use the performance budget intelligently.

Aim for a stylized but sophisticated visual identity.

Include where practical:

- atmospheric fog
- dynamic shadows
- procedural sky
- cloud layer below the island
- color grading through lighting/fog choices
- emissive materials
- subtle glow/bloom-like treatment for bright magical objects
- animated vegetation
- smooth transitions
- cinematic composition of landmarks

If full post-processing bloom is too expensive or awkward under the single-file constraint, implement a cheaper glow approximation rather than sacrificing frame rate.

---

# 14. ART DIRECTION ANCHOR

Use these fixed choices so the world has a specific identity instead of generic low-poly:

- **Look:** stylized and painterly — smooth gradients, soft rim light, restrained noise. Not flat-shaded low-poly, not photoreal.
- **Palette:** warm gold key light; teal-to-emerald vegetation; pale sand; cool violet-blue shadows; dusty rose and amber at dawn and dusk; deep indigo night. All magical/emissive elements (runes, puzzle stones, beam, fireflies) share one cyan-and-gold accent family so they read as a single system.
- **Signature landmark:** a tall ancient spire standing in the ruin, with dormant rune lines. It must be visible from the player's spawn point, dominate the skyline, and read as a clear silhouette from anywhere on the island.
- **Puzzle payoff:** solving the puzzle ignites the spire's runes in sequence and fires a beam of light from its tip into the sky, visible from across the island. You may add further effects (lighting shift, sound, ruin animation) but the beam from the spire is required.
- **Opening composition:** the player spawns near the edge of the beach or a hill, facing the spire, shortly after sunrise, with the sun placed so it side-lights or backlights the spire. Frame the spire with foreground hills or trees.

---

# ENGINEERING REQUIREMENTS

Organize the JavaScript into clearly labeled sections.

For example:

```text
1. Configuration
2. Renderer / Scene / Camera
3. Procedural Noise
4. Terrain Generation
5. Vegetation
6. Sky / Lighting / Day-Night
7. Weather
8. Water
9. Creatures
10. Physics Objects
11. Puzzle
12. Audio
13. Player Controller
14. UI / HUD
15. Game Loop
16. Cleanup / Utilities
```

Use functions/classes/modules-within-the-file where they improve clarity.

Avoid a single monolithic animation function containing all game logic.

Separate simulation updates conceptually:

```javascript
updatePlayer(dt);
updateWeather(dt);
updateCreatures(dt);
updatePhysics(dt);
updateWater(dt);
updateDayNight(dt);
updateQuality(dt);
render();
```

Use delta time correctly so simulation behavior is not frame-rate dependent.

---

# ROBUSTNESS

The page must not depend on a development server.

Assume it is opened as:

```text
file:///.../index.html
```

Avoid browser features that fail purely because the page is using the `file://` protocol.

The experience should remain functional even if audio initialization fails.

Avoid uncaught exceptions.

---

# DIAGNOSTICS

- **Error overlay:** register `window` `error` and `unhandledrejection` handlers that show a small, dismissible, readable overlay with the message and source location. Do not stop the game loop when the error is recoverable. Route GLSL compile/link failures into the same overlay so a broken shader never appears as a silent black screen: use `renderer.debug.onShaderError` if it exists in the pinned version, otherwise hook `console.error`. Wrap initialization of non-essential subsystems (audio, ambient sounds, creatures, weather particles) in `try/catch` so one failing subsystem degrades gracefully instead of killing the world.
- **`?debug` URL parameter:** when the page URL contains `?debug`, show an extra small HUD block with: the WebGL renderer string (`WEBGL_debug_renderer_info` → `UNMASKED_RENDERER_WEBGL` if available, otherwise `gl.getParameter(gl.RENDERER)`), draw calls, triangles, geometry and texture counts (`renderer.info`), smoothed frame time in ms, current render scale, and current quality tier. Without `?debug`, none of this is shown (the normal FPS counter stays). This exists so that a hybrid-GPU laptop running Chrome on the integrated GPU is visible immediately.

---

# PRIORITY ORDER

If you encounter trade-offs, prioritize in this order:

1. The page actually runs by double-clicking `index.html`
2. Stable controls and exploration
3. 60 FPS-class performance
4. Cohesive visual world
5. Day/night + atmosphere
6. Interaction and puzzle
7. Living creatures
8. Weather
9. Audio
10. Additional spectacle

Do **not** destroy performance or stability merely to tick every requirement.

A convincing implementation of a feature is preferable to a technically elaborate but unstable one.

---

# WORKING METHOD

Before writing the implementation, internally design the architecture and identify the major performance risks.

Then implement the complete solution.

Do not give me a tutorial or ask me questions.

Do not stop at pseudocode.

Do not omit sections with comments such as:

```javascript
// TODO
// implementation omitted
// add particles here
```

Deliver the **complete runnable file**.

Before finishing, audit the code line by line against this checklist and fix every violation:

- every identifier, function, and uniform that is referenced is defined once and in scope;
- every GLSL program is consistent: varyings match between vertex and fragment stages, precision is declared, all uniforms are declared and updated;
- every Three.js API used exists in r160 with the signature used;
- the performance budget rules are honored (draw-call estimate, shadow/reflection/transparency limits, light counts);
- every requirement in sections 1–14 maps to code you can point at;
- the file is complete and ends with `</html>`.

---

# RESPONSE FORMAT

If you can write files, write `index.html` to the working directory and reply with only items 1 and 3 below; do not paste the file into the reply. Otherwise, your response should contain all three items:

1. A very short architecture summary.
2. The complete contents of `index.html` in **one code block**.
3. A short final note containing:
   - what you built
   - the principal performance/engineering trade-offs
   - any scope you cut to fit the output limit
   - what you would add with substantially more development time

The HTML is the actual deliverable. Keep commentary brief.

This is a capability test.

**Make the world memorable.**
