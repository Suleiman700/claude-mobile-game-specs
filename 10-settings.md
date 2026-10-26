# 10 — Settings

> One small popup where players control sound, vibration and privacy.

## What it is

A popup (opened from the main menu and the pause menu) with the player's preferences: music and
sound volumes, vibration, and the few legal/account options the game needs (privacy, language,
restore purchases).

## Why it matters

- Players often play muted or with music off; if they can't control audio, they uninstall.
- Some options are required by law or store policy (privacy options under GDPR, restore purchases on
  iOS).
- A clean, consistent settings screen makes the game feel finished.

## When to use it

- **Every game** with sound needs at least music and sound controls.
- Add rows only for features the game actually has (notifications, language, purchases).
- Very small games can put two toggles directly in the pause menu instead of a separate popup.

## When it happens

Whenever the player opens it: gear button on the main menu, Settings in the pause menu.

## Before you build (for Claude)

1. Confirm which rows this game needs (ask if unsure: notifications? languages? purchases?).
2. Show **Defaults** and ask: *"Use these values, or change any?"* Wait.
3. **Ask the user:**
   - which optional rows to include,
   - the privacy policy URL if the game shows one,
   - default volumes if they want something other than 0.8.

## Defaults

| Row | Control | Default | Why |
|---|---|---|---|
| Music | slider `0..1` with a music icon | `0.8` | Fine control, not just on/off |
| Sound effects | slider `0..1` with a speaker icon; dragging plays a sample click | `0.8` | Hear the level while setting it |
| Vibration | ON/OFF switch (green ON, grey OFF) | `ON` | Some players hate buzzing |
| Notifications | ON/OFF switch; only if the game has reminders (`13`) | follows permission | User control |
| Language | button → list; only if the game is translated (`21`) | device language | |
| Privacy options | button, **only when the consent SDK says it's required** (`18`) | hidden otherwise | Legal requirement, no clutter |
| Restore purchases | button; only if the game sells non-consumables (`19`) | — | Required on iOS |
| Version | small muted line at the bottom | `vX.Y.Z` | Support and bug reports |

## How it works

```
        ┌──────────────┐          (✕)
  ┌─────│   SETTINGS   │────────────┐
  │  ♫ Music                        │
  │  ━━━━━━━━━━━━━━━━━━━●━━━━━       │
  │  🔊 Sound effects               │
  │  ━━━━━━━━━━━━━━━━━●━━━━━━━       │
  │  Vibration              [ ON ]  │
  │  [ Privacy options ]  (only if required)
  │            Version 1.2.0        │
  └─────────────────────────────────┘
```

- Changes apply instantly; they're saved when a slider is released and when the popup closes.
- Same popup and same look from the main menu and from pause (`03`).

## Rules

- Settings live in their own file, separate from the game save, so "New game" keeps them.
- Old on/off settings migrate to volumes (off → 0, on → default).
- Volume 0 means silent: no sounds play at all, the music player stops.
- Vibration OFF stops every haptic in the game.
- Never put gameplay options here that change difficulty or economy.

## Implementation — Godot 4

- An audio autoload holds `music_volume`, `sound_volume`, `vibration_on`, applies
  `linear_to_db(volume)` to players, and saves to `user://settings.cfg` (`ConfigFile`).
- A `SettingsPage.open(parent)` helper builds the shared popup with themed `HSlider` rows and a
  switch button; privacy row asks the ads/consent wrapper `privacy_options_required()`.

## Other engines

- Unity: AudioMixer exposed parameters for volumes, `PlayerPrefs` for settings, the same popup prefab.

## Done when

- [ ] Music slider at 0 stops music everywhere; sound slider at 0 silences every effect.
- [ ] Vibration OFF stops all haptics.
- [ ] Settings survive "New game" and an app restart.
- [ ] Privacy options appears only when required; optional rows appear only for features the game has.
