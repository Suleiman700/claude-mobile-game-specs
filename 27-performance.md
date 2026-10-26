# 27 — Performance

> Budget for the mid-range and low-end phones your players actually own.

## What it is

Performance budgets are hard limits for frame rate, draw calls, memory, size and start time,
checked on real low-end devices rather than the developer's computer.

## Why it matters

- Most mobile players use mid-range or older phones.
- Stutter, heat and battery drain lead to uninstalls and bad reviews.
- Large downloads lose installs, especially on mobile data.

## When to use it

- **Every mobile game.** 3D and particle-heavy games need it most.

## Before you build (for Claude)

1. Confirm the target devices and renderer. Ask if unsure.
2. Show **Defaults** and ask: *"Use these values, or change any?"* — wait for the answer.
3. **Ask the user:**
   - What is the lowest test device?
   - Is a 30 fps cap acceptable on low-end phones?

## Defaults

| Budget | Default | Why |
|---|---|---|
| Frame rate | stable `60 fps` mid phone, `30 fps` floor on low-end | Smooth enough everywhere |
| Renderer | Compatibility (GLES3) | Widest device support |
| Draw calls | < `150` per frame in play | Cheap GPUs |
| Shadows | one directional light, soft shadows; off on low-end | Biggest GPU cost |
| Animated characters | ≤ `25` on screen | Skinning cost |
| UI viewports (portraits, tags) | update only when visible; ≤ `10` live | Hidden render cost |
| Download size | < `150 MB` (Play base limit 200 MB) | Installs on mobile data |
| Cold start | ≤ `3 s` to menu (`08`) | First impression |
| Memory | < `500 MB` | Avoid being killed |
| Background | no processing while minimized | Battery |

## How it works

- Measure on the low-end device every milestone, not only before release.
- Pool things spawned often (coins, projectiles, customers, enemies).
- Turn off expensive features on low-end devices (shadows, MSAA, particle counts).

## Rules

- No per-frame allocations in update loops (no new styles, tweens or arrays every frame).
- Textures compressed (ETC2/ASTC on Android); no oversized source textures.
- Profile before optimizing; fix the biggest cost first.

## Implementation — Godot 4

`rendering_method.mobile = "gl_compatibility"`; VRAM compression ETC2/ASTC; SubViewports with
`UPDATE_WHEN_VISIBLE`; object pools; the remote profiler on device.

## Other engines

Unity: URP mobile settings, SRP batcher, GPU instancing, Addressables to keep the base small.

## Done when

- [ ] A 10-minute session on the low-end device holds the frame-rate floor with all systems on.
- [ ] No memory growth over the session.
- [ ] Download size and cold start are within budget.
