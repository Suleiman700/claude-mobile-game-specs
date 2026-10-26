# 04 — HUD layout (game screen)

> Where everything sits on the play screen: progress and <currency> top-left, menu buttons on the right, one big main action at the bottom, the world visible in the middle, floating labels always readable.

## What it is

The fixed arrangement of on-screen elements during play (the heads-up display), used by top
casual games: a progress indicator and <currency> counter top-left, pause top-right, a column of
square menu buttons on the right edge, the next goal and the main action button at the bottom, and
rules for labels that float over the 3D/2D world.

## Why it matters

- Players find things where they expect them; every top idle/casual game uses this arrangement.
- The middle of the screen stays free for the game itself.
- Consistent edges and sizes make the game look finished on any phone.

## When to use it

- **Tycoon / idle / restaurant / merge**: the full layout.
- **Shooter / runner / puzzle**: keep top-left (score/<currency>), top-right (pause) and safe areas;
  replace the bottom main button with the game's controls (`07`); the right column is optional.
- Skip only for games with no HUD at all (pure cinematic, menu-only).

## When it happens

Built with the first playable level; every new feature adds its button or label here.

## Before you build (for Claude)

1. Confirm the game type to pick the full or reduced layout (ask if unsure).
2. Show the sketch and **Defaults**, ask: "Use these values, or change any?" and wait.
3. **Ask the user:**
   - which right-column buttons this game has (Daily, Goals, Free <currency>, Shop, Events...);
   - what "progress" means here (level %, stars, map completion, wave);
   - the main bottom action (Upgrades, Build, Shop...).

## Defaults

| Setting | Default | Why |
|---|---|---|
| Progress | star/level icon `96` px with its number, bar `244×50` next to it | Long-term goal always visible |
| <currency> chip | `232×64`, icon overlapping the left end, `+` button at its right | Standard currency look |
| Pause | square `96`, top-right | Thumb-safe, expected spot |
| Right column | squares `104`, top at `300`, gap `14` | Doesn't cover the play area center |
| Next-goal strip | `460` wide white pill + bar; green and pulsing when affordable | Always shows what's next |
| Main button | Primary `340×108`, bottom center, `40` px above the banner | One clear action |
| Hidden during tutorial | right column + next-goal strip | Less noise on day 1 |
| Floating label margin | `20` px from screen edges and HUD buttons | Labels never cut |
| Label over player | fades to α `0.3` | Player never hidden |

## How it works

```
 (★3)[████████ 45%]                         [ II ]      ← progress + pause
 [🪙 1,250  (+)]                                         ← <currency> chip; + = how to get more
   +48                                                   ← gain/loss text (05)
 [DEV]                                                   ← debug builds only (16)
                                              [🎁 Daily]① ← right column with count badges (02)
                                              [🏆 Goals]③
                                              [▶ Free ]
                     ( game world )
          [ Next: <unlock>    85 / 130  ████░░ ]         ← next-goal strip
          ⓘ  <short hint>                                ← tip pill (only when there's a tip)
                  [ ⬆   <main action>  ]④                ← Primary, badge = affordable count
 ═══════════════════ banner ad ═══════════════════       ← (11)
```

### Floating world labels

Price tags, names, speech bubbles and markers drawn over the world:

- Clamp them inside the screen with a `20` px margin: slide the label sideways in the camera plane
  (the object stays put) when it would be cut by an edge.
- Keep them out from under HUD buttons (right column, pause, <currency> chip): treat those rects
  as edges.
- Fade a label that would cover the player (e.g. the player standing on a price tag) to α `0.3`.
- Important things off screen show an edge badge with an arrow pointing to them.
- The next-goal strip repeats the next price tag's info so nothing is lost.

## Rules

- Everything sits inside the **safe area** and above the **banner** (move the bottom group up by the
  banner height; react when the banner shows/hides).
- Every right-column and main button has a **count badge** when it holds something (`02`).
- Hide a button whose feature is unavailable (cooldown, no ad loaded) instead of disabling it. Buttons for things the
  player can't do *yet* (can't afford, locked) stay visible and disabled (`02`, "Disabled or hidden?").
- The HUD never covers the player's usual area or the main interaction spots; check screenshots with
  the player at every station.
- Two currencies: the second chip goes under the first, same size rules.

## Implementation — Godot 4

A `CanvasLayer` HUD with a full-rect `Control` root (`MOUSE_FILTER_IGNORE`) and safe-area offsets
applied to the root; the touch joystick is the first child so buttons take touches first. The bottom
group is a `VBoxContainer` anchored center-bottom, growing upward. Floating labels: a `Sprite3D` /
`Label3D` per label with a per-frame clamp using `Camera3D.unproject_position`.

## Other engines

- Unity: a Canvas with a SafeArea panel, anchors as above, a VerticalLayoutGroup for the right
  column; world labels via `Camera.WorldToScreenPoint` with the same clamp.

## Done when

- [ ] Captures at 9:16 and 9:19.5 with the banner on: nothing clipped, nothing overlapping, equal
      left/right margins, right-column squares identical.
- [ ] Badges match the screens they open.
- [ ] Floating labels near edges stay fully readable and never hide the player.
