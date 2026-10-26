# 02 — Buttons and badges

> Chunky pressable buttons in a few fixed styles, square side buttons, and red count badges that show how many things are waiting inside.

## What it is

The button kit of the game: a main button with a thick darker "lip" so it looks physical, a fixed
set of color styles by meaning, square icon buttons for the HUD side menu, round close and plus
buttons, tabs, price buttons, and **count badges**: small red bubbles on a button's corner with the
number of things waiting behind it.

## Why it matters

- Buttons that look pressable get pressed; flat text links get missed.
- Color by meaning (green = go/claim, grey = can't, purple = video) teaches players without words.
- Count badges are the strongest pull back into menus: "3 rewards waiting" gets tapped.

## When to use it

- **Always** in any game with menus or a HUD.
- Count badges: whenever a button opens something that can hold claimable or affordable things
  (upgrades, goals, daily rewards, shop, inbox). Skip them in games without such menus (pure
  arcade runner with only Play/Settings).

## When it happens

Built right after the design language (`01`); used on every screen from the first prototype on.

## Before you build (for Claude)

1. Confirm which menus the game has, to know which buttons need badges (ask if unsure).
2. Show **Defaults** and ask: "Use these values, or change any?" and wait.
3. **Ask the user:**
   - the list of HUD buttons and what each badge should count;
   - any extra button styles the game needs.

## Defaults

| Setting | Default | Why |
|---|---|---|
| Button height | `100` (main), `88–96` (rows), `72` visible in dense lists with the whole row tappable (≥ `88` tall) | Easy to hit |
| Bottom lip | `11` px in the darker shade (`4` when pressed) | Reads as a physical, pressable button |
| Rim | `3` px in the lip color | Crisp edge on any background |
| Radius | `26` | Soft |
| Icon | left of the text, `height × 0.48` | Faster recognition |
| Square side button | `104×104`, icon on top, 20 px outlined caption below | Compact HUD menu |
| Badge | red pill, `4` px white rim, 26 px white number, top-right corner, pulses | Shows "something inside" |
| Badge position | overlaps the top-right corner (`x = width − 32`, `y = −14`) | Same spot on every button |
| Badge max | `9+` | Stays small |

## How it works

### Styles (by meaning)

| Style | Use |
|---|---|
| Primary | the one main action on a screen (<main action>, Double it, Collect 2×) |
| Green | play, resume, claim, buy when affordable |
| Secondary | settings, navigation |
| Ad | anything that plays a rewarded video; always with the ▶ video icon |
| Gold | MAX, big rewards, Daily |
| Danger | destructive (reset, new game confirm) |
| Grey | can't afford, locked (padlock icon), GO |
| Neutral | Main menu, Nice!, Collect (the "no thanks" choice) |

States: normal · hover (face lightened 12%) · pressed (lip squashed, content shifts down 6 px) ·
disabled (flat grey; tapping it plays an error sound + error haptic, never silently ignored).

**Disabled or hidden?** Something the player can't do *yet* (can't afford, locked, max level) stays visible as a disabled/Grey button that explains why when tapped. A whole feature that isn't available *right now* (no ad loaded, cooldown running, switched off) is hidden, never shown disabled (`04`, `25`).

### Square side buttons

A column of squares on the right edge (`04`): icon on top, short outlined caption under it
("Daily", "Goals", "Free", "Shop", "Events"). An **Ad** square gets a small red video tag on its
top-right. When a feature is unavailable, hide its square rather than showing it disabled.

### Count badges

```
 ┌──────────┐③      ← red bubble with the count, on the top-right corner
 │  ⬆ Upgrades │
 └──────────┘
```

| Button | Badge shows |
|---|---|
| Upgrades | upgrades the player can afford **now** |
| Goals / Missions | finished goals not yet claimed |
| Daily | `!` while today's reward is unclaimed |
| Shop | affordable items |
| Inbox / Events | unread or claimable count |

### Other parts

- **Round close (✕):** 76 px, `BADGE` red, 5 px white rim, white cross, shadow; top-right of popups.
- **Plus (+):** 44 px round green with white rim (visible), inside an invisible `88×88` touch area, next to a <currency> chip; opens how to get more.
- **Tabs:** a row of neutral buttons; the selected one is Primary.
- **Price button (rows):** 160×96; Green with price when affordable, Grey with price when not,
  Gold "MAX" when maxed, Grey padlock when locked (and the row explains how to unlock).

## Rules

- A badge is visible only when its number is > 0; never show "0".
- Badges pulse gently (scale 1.14 every 1.1 s) while visible.
- Badges update every frame or on the relevant signals, so they're always true.
- Inside the opened screen, the items the badge counted are highlighted (green border, pulsing
  claim/buy button) so the player finds them immediately.
- Every button gets the press effect (`06`).
- All buttons in one row/column share height and icon size.

## Implementation — Godot 4

A static `UiKit` class: `button(text, style, icon, height)`, `square(icon, style, caption)`,
`badge() -> [panel, label]`, `close_button()`, `plus_button()`, `press_effect(button)`, `pulse(control)`.
Styles are theme type variations (`GreenButton`...) from `01`.

## Other engines

- Unity: a Button prefab per style with a 9-slice sprite (face + lip baked), an Animator for the
  press squash; a Badge prefab parented to the button's top-right anchor.

## Done when

- [ ] All buttons in a group have equal height; icons align; badges sit at the same corner offset.
- [ ] Tapping a disabled button gives sound + haptic feedback.
- [ ] Every badge number matches what the opened screen shows; badges hide at 0.
