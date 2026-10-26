# 01 — Design language

> One consistent arcade-casual look (palette tokens, font, type scale, spacing, safe area) that every screen is built from.

## What it is

The visual rules every screen in the game follows: a small set of named color **tokens**, one rounded
font, a fixed type scale, corner radii, spacing steps and screen margins. Buttons (`02`), popups
(`03`), the HUD (`04`) and everything else are built only from these pieces.

The style is the one used by top Play Store casual games (idle tycoons, restaurant games, merge and
puzzle games): chunky, rounded, bright, readable at a glance, satisfying to touch. The **structure
is fixed**; the **palette is chosen per game**.

## Why it matters

- Players judge a game in the first seconds by how it looks; a consistent, polished look reads as
  "quality" and keeps them playing.
- One palette in one place means a color change is one edit, not a hunt through every script.
- Fixed sizes and margins make every screen work on every phone shape without per-device fixes.

## When to use it

- **Always**, for any mobile game with menus and a HUD: idle/tycoon, restaurant, shooter, puzzle,
  runner, merge.
- Adjust (don't skip) for games with a strong art direction (pixel art, horror, realistic): keep the
  token system and sizes, swap the font and palette.

## When it happens

First thing in a new project, before any screen is built. Every later screen uses it.

## Before you build (for Claude)

1. Confirm the game wants the arcade-casual style (ask if the art direction is unclear).
2. Show **Defaults** and ask: "Use these values, or change any?" and wait.
3. **Ask the user:**
   - the palette: Example A, Example B, or a new one built from the game's theme;
   - the game name for the logo;
   - the font if they want something other than Fredoka.

## Defaults

| Setting | Default | Why |
|---|---|---|
| Viewport | `720×1280` portrait, stretch `canvas_items`, aspect `expand` | One layout for every phone |
| Font | `Fredoka` (OFL): weight 600 body, 700 buttons/headers/numbers | Rounded, friendly, very legible |
| Base text size | `34` px | Readable on small phones |
| Corner radius | buttons `26`, panels `40`, cards `28`, pills `26` | Soft, toy-like |
| Screen margin | `24` px sides, `40` px top/bottom (+ safe area) | Thumb room, notch room |
| Spacing scale | `4 / 8 / 12 / 16 / 24 / 32` | Consistent rhythm |
| Min touch target | `88×88` px (~44 pt) **tappable area**; a visible shape may be smaller if invisible padding makes up the rest | Fat-finger safe |

## How it works

### Palette tokens (fill per game)

Every color in the game comes from these tokens. No one-off colors.

| Token | Role | Example A (boutique hotel) | Example B (street food) |
|---|---|---|---|
| `PRIMARY` | the main call to action (<main action>, Double it) | teal `#16a69a` | orange `#ff9a1f` |
| `GREEN` | play, resume, claim, buy | `#37b24d` | `#3ec24a` |
| `SECONDARY` | settings, secondary actions | navy `#2f5fb3` | blue `#36b6ff` |
| `AD` | anything that plays a rewarded video | plum `#8e57d8` | purple `#9a6cff` |
| `GOLD` | MAX, big rewards, Daily | `#f2b632` | yellow `#ffcf3a` |
| `DANGER` | destructive actions, errors | coral `#ec5f52` | red `#ef5a3c` |
| `GREY` | can't afford, locked, GO | `#c3c9d4` | `#c9bca6` |
| `NEUTRAL` | neutral buttons | ivory `#f6f8fc` | cream `#fff6e2` |
| `PANEL` | popup body | `#f4f7fc` | `#fff4dc` |
| `CARD` | rows inside popups | `#e7eef8` | `#fbe9cc` |
| `RIBBON` | popup title ribbon | gold `#e9ae2f` | orange `#ff9a1f` |
| `CHIP` | dark HUD pills | `#14203a` α .8 | `#281e18` α .78 |
| `TEXT` | body text | navy `#1e2a44` | brown `#5c3314` |
| `GAIN` | "+X" <currency> | `#ffd43b` | `#ffd43b` |
| `LOSS` | "−X" <currency> | `#ff5a4e` | `#ff5a4e` |
| `BADGE` | count badges, close button | `#e8473b` | `#e2523b` |

Each button color also needs a **lip** (≈25% darker) and a **text outline** (≈45% darker).

### Type scale

| Use | Size | Weight | Style |
|---|---|---|---|
| Logo (`TitleLabel`) | 92 | 700 | gold fill, thick dark outline (28), drop shadow y+8 |
| Popup title (`HeaderLabel`) | 50 | 700 | white, outline 12, ALL CAPS, on the ribbon |
| Button | 40–44 | 700 | white + outline on colored; `TEXT` on neutral/grey |
| Body | 34 | 600 | `TEXT` |
| HUD numbers (`HudLabel`) | 32 | 700 | white, outline 9 |
| Small / captions | 20–26 | 600 | muted `TEXT` α .65 |

Numbers of <currency> always go through one formatter: `1,234`, `12.3K`, `4.56M`, rounded **down**
(never shows more than the player has).

## Rules

- Text on colored backgrounds always has an outline (contrast ≥ 3:1 even on bright colors).
- One primary action per screen; it's the biggest, most colorful button.
- Nothing touches the screen edge; everything respects the safe area (notch, home bar, rounded corners).
- Icons are white and tinted in code (one icon file serves every color).
- No more than ~4 font sizes on one screen.
- No color appears in UI code that isn't a token.

## Implementation — Godot 4

- Generate `ui/theme.tres` from one script (e.g. `tools/make_theme.py`) so the palette lives in one
  place: button variations `PrimaryButton`, `GreenButton`, `SecondaryButton`, `AdButton`,
  `GoldButton`, `DangerButton`, `GreyButton`; panel types `PanelContainer`, `CardPanel`,
  `ReadyCardPanel`, `RibbonPanel`, `ChipPanel`; label types `TitleLabel`, `HeaderLabel`, `HudLabel`,
  `ToastLabel`; `HSlider`, `ProgressBar`.
- Set `gui/theme/custom = "res://ui/theme.tres"` in `project.godot`.
- Fonts as `FontVariation` resources (`font_regular.tres` weight 600, `font_bold.tres` 700).
- On the first import the theme may load before the font is imported; run `--import` twice in CI.

## Other engines

- Unity: a ScriptableObject palette, a TextMeshPro font asset with an outline material, 9-slice
  sprites for buttons/panels generated from the palette.
- Web: CSS custom properties per token.

## Done when

- [ ] Every color on every screen maps to a token; no stray hex values in UI code.
- [ ] Screens captured at 9:16 and 9:19.5 pass `17-testing-and-screenshots.md` (alignment, contrast, safe area).
- [ ] Changing one token in the theme script recolors the whole game.
