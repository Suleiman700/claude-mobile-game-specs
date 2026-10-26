# 07 — Touch controls

> Mobile games need visible on-screen touch controls: a floating joystick with an idle hint, plus action buttons when the game needs them.

## What it is

The on-screen controls the player uses on a phone: a **floating joystick** (touch anywhere, drag to
move) for movement, and round **action buttons** (jump, dash, attack, interact) for games that need
them. Keyboard and mouse remain as a fallback for desktop and testing.

## Why it matters

- A mobile game that only works with a keyboard, or whose controls are invisible, feels broken.
- Players must see at a glance how to move; an invisible joystick makes new players think the game
  is stuck.
- Good touch controls (analog, dead zone, release on focus loss) prevent frustration and accidental
  movement.

## When to use it

- **Any mobile game where the player moves a character or vehicle**: tycoon/idle with a walking
  player, restaurant, shooter, top-down adventure. Runners usually use swipes instead.
- Action buttons: shooters, platformers, brawlers. Idle/tycoon games usually need only movement
  (actions happen by standing in zones).
- Skip for tap-only games (puzzle, merge, card games).

## When it happens

From the first playable build. The first-session hint shows until the player has moved a little.

## Before you build (for Claude)

1. Confirm how the player moves (joystick, swipe, tap-to-move, none) and ask if unsure.
2. Show **Defaults** and ask: "Use these values, or change any?" and wait.
3. **Ask the user:**
   - which actions the game needs besides moving (jump, dash, attack, interact);
   - whether the joystick should be floating (anywhere) or fixed (bottom-left).

## Defaults

| Setting | Default | Why |
|---|---|---|
| Movement | floating joystick: touch anywhere (outside buttons) to place it, drag to move | Works for any hand and thumb position |
| Radius | `70–110` px knob travel | Comfortable thumb drag |
| Dead zone | `8%` of radius | No drift from shaky fingers |
| Output | length `0..1` (analog speed), screen up = world forward | Slow, precise walking possible |
| Idle hint | faint ring + knob in the lower middle of the screen (α `0.35`) while not touching | Players see there is a control |
| First-session hint | "Drag to move" label **above** the ring with the knob sliding left-right, until the player has moved `3 s` total | Teaches without text walls |
| Active look | ring α `0.25` fill + white outline `5` px, solid white knob | Clear under the thumb |
| Keyboard fallback | WASD + arrows (desktop, web, tests) | Development and PC |
| Action buttons (if any) | round `140` px, bottom-right, icon only, `16` px apart | Thumb reach |

## How it works

```
                    Drag to move          ← hint above the ring (first session only)
                     (   ◯   )            ← idle ring; the knob slides to show "drag"
          ⓘ <tip pill>                    ← HUD elements stay below, never covering the ring
                [ <main action> ]
```

1. Finger down anywhere that isn't a button → the joystick appears under the finger.
2. Drag → output follows, clamped to the radius, zero inside the dead zone.
3. Finger up → the joystick hides, output resets, the idle ring returns.

## Rules

- The joystick reads input **after** the UI, so buttons always win taps.
- It never starts on top of a button, and hides its idle hint while a popup is open.
- Release on app pause / focus loss (the OS can swallow the touch-up), and when a popup opens or the
  game pauses, so the player doesn't keep walking.
- Multi-touch: one finger moves; a second finger can press action buttons.
- Mouse emulates touch on desktop (`emulate_touch_from_mouse = true`) for testing.
- Place the idle ring and hint where no HUD element (tip pill, bottom buttons) covers them.
- A tutorial's first step waits for the first movement (`23`).

## Implementation — Godot 4

A `TouchJoystick extends Control` (full rect via `set_anchors_and_offsets_preset`; input in
`_gui_input` behind the buttons, or `_unhandled_input`), handling `InputEventScreenTouch` /
`InputEventScreenDrag`, drawing in `_draw()`, exposing `value`; releases on
`NOTIFICATION_APPLICATION_PAUSED` / `NOTIFICATION_APPLICATION_FOCUS_OUT`. The player reads the
joystick and `Input.get_vector("ui_left", "ui_right", "ui_up", "ui_down")`.

## Other engines

- Unity: a floating joystick on a full-screen invisible panel (Input System On-Screen Stick or a
  joystick asset); EventSystem ordering so buttons take priority.

## Done when

- [ ] On a phone (or touch-emulated desktop) the idle ring and first-session hint are visible.
- [ ] Dragging moves the player in the right direction at analog speed; tapping any HUD button
      never moves the player.
- [ ] Switching apps mid-drag stops the player.
