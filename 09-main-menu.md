# 09 — Main menu

> The title screen: show the game off, get the player playing in one tap.

## What it is

The screen after the splash: the game's logo over a live view of the game world, a big Play (or
Continue) button, and a few secondary buttons (Settings, New game, optionally More Games).

## Why it matters

- It sells the game every time it opens: a living world behind the logo says "this is fun" more
  than any drawing.
- One clear Play button gets players into the game fast; clutter here slows every session.
- It's where returning players decide to continue; a broken or confusing menu loses them.

## When to use it

- Almost every mobile game with saved progress (tycoon, idle, simulation, RPG, puzzle with levels).
- Skip it (go straight into play) for hyper-casual one-tap games; put Settings in the pause menu.
- Games with many levels/worlds may add a level-select screen after Play; the menu stays simple.

## When it happens

After the splash (`08`) and consent (`18`), on every app start, and when the player chooses
"Main menu" from Pause.

## Before you build (for Claude)

1. Confirm the game needs a main menu (ask if unsure).
2. Show the sketch and **Defaults** and ask: *"Use these values, or change any?"* Wait.
3. **Ask the user:**
   - the logo text (one or two lines) and the game's palette (`01`),
   - **whether they want a "More Games" button**, and if yes, their store developer page URL,
   - any extra buttons (Shop, Levels, Leaderboard).

## Defaults

| Setting | Default | Why |
|---|---|---|
| Backdrop | the real game world (fully built, a few characters idling), camera slowly orbiting, `90 s` per circle | Shows the game, feels alive |
| Readability | dark gradient at the top (behind the logo) and bottom (behind the buttons) | Text stays readable over a busy scene |
| Logo | the game font, gold fill, thick dark outline, gentle bob | Bold, on-brand title |
| Play button | Green, `600×124`, pulses; label `Play` for a fresh save, `Continue` with progress | One obvious action, honest label |
| Settings | Secondary color with gear icon, opens the shared Settings popup (`10`) | Same everywhere |
| New game | Neutral, only when a save exists; first tap turns red "Tap again", second tap resets | No accidental wipes |
| More Games | `off` — **ask the user before adding it**; when on: Ad/Plum color, opens the studio's store developer page, e.g. `https://play.google.com/store/apps/developer?id=<YourStudio>` | Cross-promotion only if the user wants it |
| Quit button | `none` on mobile | Phones don't quit apps that way |
| Money / stats | `not shown` | Money belongs in the game screen |
| Banner ad | on (see `11`); all buttons sit above it | Predictable layout |
| DEV button | top-left, debug builds only (`16`) | Testing |

## How it works

```
 [DEV]                                      ← debug builds only
              Game                          ← logo, two lines, gentle bob
              Name
     ( live 3D world, camera slowly circling )
     [ ▶          Play / Continue         ] ← Green, pulses
     [ ⚙ Settings ]   [ ↺ New game ]        ← side by side (New game only with a save)
     [ +         More Games               ] ← optional, only if the user said yes
                                    v1.2.0
 ══════════════════ banner ad ══════════════════
```

- Play loads the game scene (or the level select). Continue resumes the saved game.
- New game keeps settings and purchases; it only wipes progress (`22`).
- More Games opens the https store link; on Android the Play Store app handles it.

## Rules

- Never draw a 2D picture of the world when the real 3D (or 2D) world can be shown.
- The logo never covers the buttons; buttons never sit under the banner.
- Keep store links in one helper (developer page; this game's own page for the rate-prompt
  fallback, `12`).
- Android Back on the menu: "Exit game?" confirm or minimize (see `24`).
- At most 4 buttons; extra features go inside the game.

## Implementation — Godot 4

- `MainMenu extends Control` (full rect) with a `MenuBackdrop extends Node3D` child that builds the
  level's static world, adds a few idle characters and shares the game's lighting function; the
  camera orbits a focus point in `_process`.
- UI on top: logo `Label` (title theme type), a bottom `VBoxContainer` offset by the banner height.
- `StoreLinks.open_more_games()` → `OS.shell_open(DEVELOPER_PAGE)` (only if the user wants it).

## Other engines

- Unity: a menu scene that loads the level environment additively with a dolly/orbit camera; UI
  canvas on top; `Application.OpenURL` for store links.

## Done when

- [ ] Fresh save shows Play (no New game); a save with progress shows Continue + New game.
- [ ] The world moves behind the logo; logo and buttons readable at 9:16 and 9:19.5 with the banner.
- [ ] New game asks twice and keeps settings.
- [ ] More Games exists only if the user asked for it and opens their developer page.
