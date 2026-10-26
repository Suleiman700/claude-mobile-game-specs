# 28 — Upgrades and unlocks

> Show what every purchase is, what it changes, how far it goes and what it costs, at a glance.

## What it is

- **Upgrades** raise the level of something the player already owns (walk speed, carry
  capacity, weapon damage, worker speed, shop price).
- **Unlocks** add something new (a worker, a room/station/area, a weapon, a character, a level).
- The **upgrades screen** lists upgrades; **unlocks** happen in the world (floor price tags in
  tycoon games) or in a shop list, and are celebrated with a reveal popup.

## Why it matters

- Spending is the core reward loop in idle, tycoon, shooter and RPG games.
- If players can't tell what a purchase does, they stop buying and stop progressing.
- Confusing locked items ("why can't I buy this?") are a top support question.

## When to use it

- Idle, tycoon, management, shooter/RPG upgrade shops, runner upgrades, tower defense.
- The **floor price tag** part applies only to tycoon / arcade-idle games where the player buys
  by standing on a spot; other genres buy from menus.
- Skip for games without persistent progression.

## When it happens

- The upgrades screen opens from the main bottom button (`04`); its badge shows how many
  upgrades are affordable now.
- Unlocks appear one or two at a time as the player progresses; each new kind gets a reveal popup.

## Before you build (for Claude)

1. Confirm the game has upgrades and/or unlocks, and how players buy unlocks (floor tags or menu).
   Ask if unsure.
2. Show **Defaults** and the row sketch; ask: *"Use these values, or change any?"* — wait.
3. **Ask the user:**
   - The tabs (e.g. You / Base / Crew, or Player / Weapons / Gear).
   - The list of upgrades: name, max level, the stats each level changes.
   - The unlock order and how each unlock is earned.

## Defaults

| Setting | Default | Why |
|---|---|---|
| Surface | bottom sheet (`03`), ribbon "UPGRADES", ✕, tabs | Keeps the world visible |
| Picture | the actual item: live 3D portrait/model (idle animation) for characters, render or icon for objects, in a round disc `112` | Recognizable |
| Name | bold `32`, trims with … ; chip "N hired/owned" when relevant | |
| Change lines | **each stat on its own line**: `Stat current → next`, the new value in green; only stats that change | Never two stats on one line |
| Level | pips (`22×10`, green = bought) + `Lv x/max` | Shows how far it goes |
| Price button | `160×96`: Green with price (affordable) · Grey with price (not affordable) · Gold `MAX` (max level) · Grey padlock (locked) | Price always visible |
| Locked row | faded picture + how to unlock ("Unlocks after Level 3", "Buy its price tag ($200)") | Never just "locked" |
| After buying | row refreshes in place, values tick up, confirmation sound, medium haptic, `−$X` (`05`) | Clear feedback |
| Badge | main button badge = affordable rows | Draws players back |
| Reveal popup | first unlock of each kind (`03`) | Celebrates new things |

## How it works

```
 ┌──────────────────────────────────────────────────────────────────┐
 │ ( picture )  Mechanics                        [ 2 hired ]        │  ← name + chip
 │  disc 112    ┌─────────────────────────┐                ┌──────┐ │
 │  live model  │ Repair 100% → 120%      │                │ $60  │ │  ← one stat per line
 │  or icon     │ Walk 100% → 111%        │                └──────┘ │  ← price button
 │              └─────────────────────────┘                          │
 │              ▬▬▭▭▭  Lv 2/5                                       │  ← level pips
 │              Next hire: 3rd Mechanic · unlocks after Garage 2    │  ← workers only
 └──────────────────────────────────────────────────────────────────┘
```

### Workers: hires vs levels

`MAX` on a worker row means the **upgrade levels** are maxed, not that no more workers can be
hired. Every worker row shows a **"Next hire"** line saying how to get the next one ("Next
hire: 2nd Chef · unlocks after Kitchen 2" or "· buy its price tag ($500)"), or "All chefs hired".

### Unlock reveal

When something new is unlocked: popup with the item's picture, "<Name> unlocked!" and one line of
benefit ("Serves customers at the counter."), **Nice!** button. Not for repeats or upgrades.

### Floor price tags (tycoon / arcade-idle only)

| Setting | Default |
|---|---|
| Tag | dark rounded chip above the spot: picture, name, **remaining** price in gold |
| Paying | stand on it: whole price in `1.4 s`; bills fly from the player to the spot; `−$X` grows (`05`) |
| Fill | the floor ring fills **and the chip fills left→right in green**; progress is kept if the player steps off |
| Player on it | tag fades to α `0.3` so it never hides the player |
| Visible | only the next `2` tags |
| On screen | the tag slides sideways to stay fully on screen and out from under HUD buttons (`04`) |

## Rules

- Values in the player's terms: `Carry 4 → 5`, `Speed 100% → 112%`, `Damage 10 → 12`, `$16 → $21`.
- MAX rows show the final values in a gold pill instead of a change line.
- Keep a stable order inside a tab (no jumping rows).
- The sheet never grows wider than the screen: long text wraps inside the row.
- Every locked thing names its unlock condition; players never guess.

## Implementation — Godot 4

- Data list of upgrades (`id`, `tab`, `name`, `role`, `max`) and `upgrade_value(id, level)` returning
  stats joined by ` · `; the row splits them, one per line, with BBCode for the green value.
- An `UpgradesSheet` Control (tabs, rows rebuilt when affordability changes).
- A `Portrait` (SubViewportContainer + own World3D + camera) for live character pictures.
- Floor tags: a SubViewport rendering a 2D chip onto a billboard Sprite3D; a ProgressBar child
  for the fill; screen clamping by unprojecting the tag's edges each frame.

## Other engines

Unity: a ScrollRect of row prefabs bound to ScriptableObject upgrade data; RenderTexture
portraits; world-space canvas tags with a clamp script.

## Done when

- [ ] Every row shows picture, name, one stat per line, level and a price button.
- [ ] Locked rows and worker rows explain how to get more.
- [ ] Nothing overflows at 9:19.5.
- [ ] (Tycoon) tags fill while paying, fade under the player, never get cut at the edge.
- [ ] First unlock of each kind shows the reveal popup.
