# 26 — Accessibility

> Make the game readable and playable for more people with a few cheap rules.

## What it is

Accessibility means players with weaker eyesight, color blindness, motor difficulties or
sensitivity to motion and sound can still play comfortably.

## Why it matters

- About 1 in 12 men is color-blind; small text and low contrast hurt everyone on a bright screen
  outdoors.
- Players who can't read a state or hit a button simply quit.
- Store featuring favors accessible games.

## When to use it

- **Every game.** The rules below cost almost nothing when followed from the start.

## Before you build (for Claude)

1. Confirm the audience (kids, older players, everyone). Ask if unsure.
2. Show **Defaults** and ask: *"Use these values, or change any?"* — wait for the answer.
3. **Ask the user:**
   - Should there be a "Large text" option?
   - Any one-hand or no-timer requirements?

## Defaults

| Setting | Default | Why |
|---|---|---|
| Min text | `20` px canvas for captions, `26+` for anything important | Readable on small phones |
| Contrast | body ≥ `4.5:1`, large/outlined ≥ `3:1` | Readable outdoors |
| Color-blind safety | state never by color alone: add icon/text (padlock = locked, check = done, "MAX") | Works for everyone |
| Touch targets | ≥ `88×88` px tappable area (visible shape may be smaller with invisible padding) | Motor-friendly |
| Haptics / sounds | always optional in Settings (`10`) | Sensory comfort |
| Motion | no flashing > 3 times/s; screen shake small and short | Photosensitivity |
| Large text | `optional` switch scaling UI text 115% | Low vision |
| Timers | show numbers, not just a shrinking bar | Clear without color |

## How it works

- All sizes come from the theme, so one scale value can grow all text.
- Every state (affordable, locked, claimable, maxed) has a shape or word, not only a color.
- Check screenshots in grayscale: if a state disappears, add an icon or text.

## Rules

- No essential information only in color or only in sound.
- No rapid flashing; confetti and pulses are gentle.
- Tappable areas never smaller than the minimum target, even in dense lists; a smaller visible shape needs invisible padding (`01`).

## Implementation — Godot 4

Theme-level font sizes (one place to scale); a grayscale check on captures
(`magick in.png -colorspace Gray out.png`); Settings switch for large text applying a theme
scale.

## Other engines

Unity: same rules; TextMeshPro sizes from one style sheet; screen reader plugin for menus if needed.

## Done when

- [ ] Grayscale captures still show locked / claimable / maxed clearly.
- [ ] No text under 20 px; every tappable area ≥ 88×88 px.
- [ ] Sounds, music and vibration can each be turned off.
