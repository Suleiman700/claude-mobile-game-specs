# 16 — Dev panel (DEV button)

> A hidden-in-release DEV button that opens shortcuts to test every system in seconds.

## What it is

A small purple **DEV** button in the top-left corner, visible only in development builds. It opens a
popup full of shortcut buttons: add money, unlock everything, spawn special characters, jump the
daily reward to day 7, change game speed, reset the save, and so on.

## Why it matters

Testing a mobile game by playing it normally is slow: reaching day 7 of a daily reward takes a week,
reaching late-game upgrades takes hours. The dev panel lets you (and Claude) check any state in
seconds, reproduce bugs, take screenshots of rare screens, and verify fixes. It pays for itself on
the first day.

## When to use it

- **Every game** that has progression, currencies, timers, popups or random events.
- Skip only for tiny prototypes with no state.
- It must **never** reach players: release builds don't contain it.

## When it happens

Whenever a developer taps DEV: on the main menu (save-level tools only) and in-game (all tools).

## Before you build (for Claude)

1. Confirm the game is past the prototype stage (it has a save or progression); ask if unsure.
2. Show **Defaults** and ask: *"Use these values, or change any?"* Wait for the answer.
3. **Ask the user:**
   - which game systems exist (propose one section per system from the template below);
   - any extra shortcuts they test often.

## Defaults

| Setting | Default | Why |
|---|---|---|
| Button | small purple "DEV", `96×56` visible, `96×88` tappable, top-left (under the money counter in game) | Easy to find, out of the way |
| Visible when | debug builds only (`OS.is_debug_build()`) | Never in release |
| Panel | standard popup (`03`), width `660`, scrolling | Same look as the game |
| Layout | sections with a title and 2-column small buttons (height `72`) | Many actions, compact |
| Status line | money, unlocks x/y, daily state, game speed | See the effect instantly |
| After each action | save the game, refresh the status line | State always consistent |
| Danger actions | red, in a "Danger" section at the bottom | No accidents |

## How it works

### Section template (adapt to the game)

| Section | Buttons |
|---|---|
| Money | +100, +1,000, +10,000, +1M, money to 0 (second currency the same) |
| Progress | unlock next, unlock all, max all upgrades, reset upgrades, lock everything / next level, win level |
| Characters | spawn normal customer/enemy, spawn special ×2 / ×3, fill the queue/wave, make all leave |
| World states | game-specific: tables dirty, machines broken, all clean, trigger an event |
| Gifts, ads & popups | drop a gift, free-cash ready (reset cooldown), welcome back, full-screen ad, reveal popup, toast |
| Daily reward | reset streak, missed a day, claim again today, **Day 1…7 ready**, open daily |
| Goals | complete all, reset goals, open goals |
| Tutorial | restart tutorial, skip tutorial |
| Notifications / rating | show the soft-ask, show the rate prompt, schedule a test notification in 10 s |
| Time | speed ×1, ×2, ×5, ×10 |
| Danger | reset whole game (back to the main menu) |

On the main menu only save-level sections show (no level is loaded).

```
 ┌──────────── DEVELOPER ────────────┐ (✕)
 │ Money $1,234 · unlocks 4/12 · daily: day 3 ready · ×1 │
 │ Money                              │
 │ [ + $100 ] [ + $1,000 ]            │
 │ [ +$10,000 ] [ + $1M ]             │
 │ Daily reward                       │
 │ [ Day 1 ready ] [ Day 2 ready ] …  │
 │ Danger                             │
 │ [ Reset whole game ]               │
 └────────────────────────────────────┘
```

## Rules

- Never ship it: check an exported **release** build has no DEV button.
- Every new system adds its DEV shortcuts in the same change.
- Destructive actions are red and grouped in "Danger".
- A test presses DEV buttons programmatically and checks the effect.
- The panel uses the game's normal UI kit, so it also tests the UI.

## Implementation — Godot 4

- `DevPanel.button(open_callable) -> Button` and `DevPanel.open(parent, level_or_null)`.
- Sections built from specs: `[label, style, action_callable]`.
- Level helpers it calls: `spawn_customer(special_mult)`, `dev_unlock_next()`, `drop_gift()`.
- Speed via `Engine.time_scale`.
- Only created when `OS.is_debug_build()`.

## Other engines

Unity: a canvas prefab inside `#if DEVELOPMENT_BUILD || UNITY_EDITOR`, built from the same section
specs; time via `Time.timeScale`.

## Done when

- [ ] DEV shows in debug builds and is absent in a release export.
- [ ] Every section's buttons work in a running debug build.
- [ ] A test presses at least: add money, Day 7 ready, missed a day, unlock next, max upgrades,
      complete goals, and checks each result.
