# 08 — Splash screen

> The first second of the game: brand, load, and get the player to the main menu fast.

## What it is

Everything the player sees between tapping the app icon and the main menu: the engine's boot image,
an optional short studio logo, and a loading step while the menu (and its 3D backdrop) loads.

## Why it matters

- It's the first impression; a black or white flash, a stretched image or a frozen screen makes the
  game feel cheap before it starts.
- Slow starts lose players: every extra second before the menu costs installs that never play.
- It's the right moment for things that must happen before anything else (privacy consent, language
  detection, save loading).

## When to use it

- **Every mobile game** needs at least a proper boot image (never the engine's default logo).
- The studio logo step is optional; skip it for hyper-casual games where speed is everything.
- A separate loading screen with a progress bar is only needed when loading takes over ~1 s
  (3D menus, big asset sets). Tiny 2D games can go straight to the menu.

## When it happens

1. App launch → engine boot image (static, shown by the engine itself).
2. Studio logo (optional, ~1 s, tap to skip).
3. Loading of the main menu in the background (progress bar if slow).
4. Privacy consent form if required (see `18`), then the main menu (`09`).

## Before you build (for Claude)

1. Confirm the game needs more than the boot image (ask if unsure).
2. Show **Defaults** and ask: *"Use these values, or change any?"* Wait for the answer.
3. **Ask the user:**
   - the studio logo file (or "no studio logo"),
   - the game logo / icon art and the brand background color,
   - the slowest phone they care about (to judge load time).

## Defaults

| Setting | Default | Why |
|---|---|---|
| Boot image | the game's own portrait splash (logo centered on the brand color), no engine logo | First frame already looks like the game |
| Boot image scaling | not stretched (`fullsize` off), background filled with the brand color | Looks right on every aspect ratio |
| Studio logo | `1.2 s` fade in / hold / fade out, skippable by tap | Brand without wasting time |
| Loading | load the main menu in the background; show a progress bar only if it takes > `1 s` | No frozen screen |
| Max time to menu | `≤ 3 s` on a mid-range phone | Players leave slow starts |
| Sound | `none` on the splash; music starts on the main menu | People open games in public |
| Consent | privacy consent form (`18`) after the splash, before any ad loads | Required order |
| First launch | language auto-detected from the device (`21`), no setup screen | Zero friction |
| Load failure | friendly message + Retry button | Never a black screen |

## How it works

```
 tap icon ─► [ boot image: logo on brand color ] ─► [ studio logo 1.2 s ] ─► [ loading… ▓▓▓░ ]
                 (engine, static)                     (tap to skip)            (only if slow)
                                                                                    │
                          privacy consent (if required) ◄───────────────────────────┘
                                     │
                                     ▼
                                main menu (09)
```

- The loading step requests the menu scene in a background thread and polls progress each frame.
- As soon as loading reaches 100 % and the logo time is over, switch scenes with a short fade.
- Heavy one-time work (save load, settings, ads SDK init) starts here, in parallel with loading.

## Rules

- No white or black flash between boot image, logo and menu: same background color, or fades.
- The splash image is exported for portrait at the target aspect, never stretched.
- Never block the splash on the network; offline must start just as fast.
- Android adaptive icon: foreground logo layer + plain background layer; the boot image matches the
  icon's colors.
- The splash never shows ads.

## Implementation — Godot 4

- Project Settings: `application/boot_splash/image` = portrait splash PNG,
  `boot_splash/fullsize = false`, `boot_splash/bg_color` = brand color, `show_image = true`.
- A small `boot.tscn` as the main scene: shows the studio logo, calls
  `ResourceLoader.load_threaded_request(menu_path)`, polls `load_threaded_get_status()` for the
  progress bar, then `get_tree().change_scene_to_packed(...)`.
- On load error: show a retry label instead of hanging.

## Other engines

- Unity: Player Settings splash (custom image; remove the engine logo if the license allows), a Boot
  scene that loads the menu with `SceneManager.LoadSceneAsync`.
- Native/other: a launch screen (Android SplashScreen API, iOS LaunchScreen storyboard) + async load.

## Done when

- [ ] Cold start to an interactive main menu in ≤ 3 s on a mid-range phone.
- [ ] No white/black flash anywhere between icon tap and menu.
- [ ] Splash looks right at 9:16 and 9:19.5 (no stretching, no cut logo).
- [ ] Starting offline works; a forced load error shows a Retry message.
