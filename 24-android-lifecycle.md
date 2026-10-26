# 24 — Android lifecycle and the Back button

> Behave like a good Android app: Back closes things, minimizing saves and pauses.

## What it is

Android apps are paused, resumed, interrupted by calls and killed for memory at any moment, and
every phone has a system Back action (button or gesture). The lifecycle rules say what the game
does at each of these moments.

## Why it matters

- A Back that instantly quits the game loses players' progress and patience.
- Not saving on minimize means lost progress when Android kills the app in the background.
- A character that keeps walking after the player switches apps feels broken.
- Google Play reviewers and players expect standard Back behavior.

## When to use it

- **Every Android game.** iOS has no Back button, but the pause/resume rules apply there too.

## When it happens

- **Back:** any time.
- **Pause/resume:** home button, app switcher, phone call, screen off, full-screen ads.

## Before you build (for Claude)

1. Confirm the target platforms (Android, iOS, both). Ask if unsure.
2. Show **Defaults** and ask: *"Use these values, or change any?"* — wait for the answer.
3. **Ask the user:**
   - On the main menu, should Back ask "Exit game?" or just minimize?
   - Should the game show the pause menu after a long time in the background?

## Defaults

| Event | Behavior | Why |
|---|---|---|
| Back in play | closes the top popup/sheet; with none open, opens Pause | Standard, never loses the game |
| Back in Pause | resumes | Expected |
| Back on main menu | `"Exit game?"` confirm (Exit / Stay) | No accidental exits |
| Minimize / screen off / call | save, pause, release touch controls, mute music | Safe |
| Resume | unpause only if no popup was open; reload ads; re-plan notifications | Fresh state |
| Long background (`> 5 min`) | show Pause on resume | Never drop the player into action |
| Low memory warning | save | Android may kill the app next |

## How it works

```
 Back ─▶ popup open? ──yes──▶ close it
             │ no
             ▼
        in play? ──yes──▶ open Pause ──Back──▶ resume
             │ no (menu)
             ▼
        "Exit game?" ──Exit──▶ quit
```

- Full-screen ads pause the app on Android. Ignore those pauses for "paused by the user" logic,
  and pay ad rewards only after the ad closes (`11`).

## Rules

- The game handles Back itself; the engine must not auto-quit on Back.
- Every popup closes with Back the same way as with its ✕ (popups without ✕ ignore Back).
- Never resume straight into active play after the app was in the background a long time.
- Always save before the app can be killed.

## Implementation — Godot 4

- `application/config/quit_on_go_back = false`.
- Handle `NOTIFICATION_WM_GO_BACK_REQUEST` in the HUD (close top popup → pause) and the menu
  (exit confirm).
- Handle `NOTIFICATION_APPLICATION_PAUSED` / `RESUMED` / `FOCUS_OUT`: save, pause, release the
  joystick, mute music; on resume reload ads.

## Other engines

Unity: `OnApplicationPause`, `OnApplicationFocus`, `Input.GetKeyDown(KeyCode.Escape)` for Back
(or the Input System's Back action).

## Done when

- [ ] On a device, Back closes popups one by one, then pauses, then (on the menu) asks to exit.
- [ ] Minimizing mid-movement stops the character and saves.
- [ ] A phone call during play pauses the game.
- [ ] After a full-screen ad, rewards arrive and the game is not stuck paused.
