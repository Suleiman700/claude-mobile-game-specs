# 06 — Feel, sound and haptics ("juice")

> Every tap moves, sounds and vibrates; rewards get bigger effects; nothing feels dead.

## What it is

The small motions, sounds and vibrations that make the game feel responsive: buttons squash when
pressed, badges pulse, coins clink, rewards burst with confetti, disabled taps buzz. Together they
are often called "juice".

## Why it matters

- Feedback on every touch makes a game feel high-quality and alive; missing feedback feels broken.
- Sound and vibration confirm actions without the player having to look.
- Bigger effects for bigger moments make rewards memorable.

## When to use it

- **Always**, every genre.
- Tone it down (shorter, softer) for calm games (puzzle, relaxing idle); keep it punchy for arcade,
  shooter and runner games.

## When it happens

Built together with the button kit (`02`); every new feature adds its sounds and effects in the
same change.

## Before you build (for Claude)

1. Confirm the game's tone (calm or punchy) and ask if unsure.
2. Show **Defaults** and ask: "Use these values, or change any?" and wait.
3. **Ask the user:**
   - sound pack preference (default: free CC0 interface/casino packs + a few synthesized effects);
   - music style for the menu and play.

## Defaults

| Moment | Motion | Sound | Haptic |
|---|---|---|---|
| Button down / up | scale `0.93` in `0.06 s`, back to `1` in `0.22 s` (back-ease) | soft `select` | tick `10 ms` (neutral) / light `20 ms` (colored) |
| Disabled button tap | small shake | `error` | double buzz `30+30 ms` |
| Popup opens | pop-in (`03`) | soft `pop` (no whoosh) | — |
| Coins land | chip pop | coin tick, pitch rising | — |
| Buy / upgrade | row refresh, badge recount | `confirmation` | medium `40 ms` |
| Claim a reward | coins fly from the button; confetti on big ones | `kaching` | medium |
| Unlock something new | reveal popup (`03`) | `happy` jingle | medium |
| New level / star | confetti burst | `star` | medium |
| Badges / claimable buttons | heartbeat pulse `1.14×`, `1.1 s` | — | — |
| Failure (lost, customer leaves) | red flash on the relevant counter | `sad` / error, quiet | light |
| Background | — | music loop, ambient sounds | — |

## How it works

- Each button gets one press effect function: squash on down, spring back on up, sound, haptic.
- A sound manager plays effects by name from a small pool of players, with random pitch (`±6%`) and a
  minimum gap per sound (`40 ms`) so fast streams don't turn into noise.
- Haptics go through one function that checks the Vibration setting and the platform.
- Confetti is one particle burst in palette colors.

## Rules

- Every tap gives feedback (motion + sound + haptic); nothing feels dead.
- Frequent sounds are short and soft; long/loud sounds only for rare big moments.
- Haptics only on phones and only when Vibration is on (`10`); Android needs the `VIBRATE`
  permission.
- Music loops seamlessly; it starts on the main menu, never on the splash.
- Settings volume and vibration switches are respected everywhere.

## Implementation — Godot 4

A `Sfx` autoload: `play(key, volume_db, pitch, min_gap)` with a pool of ~10 `AudioStreamPlayer`s,
volumes from settings (`linear_to_db`), `vibrate(ms)` → `Input.vibrate_handheld`. `UiKit.press_effect`,
`UiKit.pulse`, `UiKit.confetti` (a one-shot `CPUParticles2D`).

## Other engines

- Unity: an AudioManager with pooled AudioSources and an AudioMixer; `Handheld.Vibrate` or a haptics
  plugin for durations; DOTween for squash and pulse.

## Done when

- [ ] Every button has press motion + sound; disabled taps buzz.
- [ ] Rapid coin streams sound pleasant, not droning.
- [ ] Toggling sounds/vibration in Settings is respected everywhere.
