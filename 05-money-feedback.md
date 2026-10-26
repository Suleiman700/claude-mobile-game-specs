# 05 — Money feedback (earning and spending)

> Earning flies coins into the counter with a gold "+X" rising; spending shows a red "−X" falling; automatic income stays quiet.

## What it is

The animations, texts and sounds that play whenever the player's <currency> changes: coins flying
into the counter, the counter ticking up, a gold `+X` under it on gains, a red `−X` under it on
spends, and a quieter version for automatic income (workers, idle generators).

## Why it matters

- Getting <currency> is the core reward of most casual games; making it *feel* good keeps players
  playing.
- Players should always see **how much** changed and **where** it came from or went.
- A clear `−X` stops "where did my money go?" confusion after buying.

## When to use it

- **Any game with a currency**: idle/tycoon, restaurant, shooter with coins, runner with coins,
  puzzle with gems/stars.
- Skip only for games with no currency at all.

## When it happens

Every time the balance changes: picking up <currency> in the world, claiming any reward, buying
anything (upgrades, unlocks, shop items, floor price tags), and automatic income.

## Before you build (for Claude)

1. Confirm the game's currencies (ask if there's more than one).
2. Show **Defaults** and ask: "Use these values, or change any?" and wait.
3. **Ask the user:**
   - the currency icon (coin, cash, gem...);
   - which income is "automatic" (workers, idle generators, offline) and should stay quiet.

## Defaults

| Setting | Default | Why |
|---|---|---|
| Earn: flying coins | `1–12` coins (more for bigger amounts), arc into the <currency> chip, stagger `0.045 s`, flight `0.5 s` | Satisfying, shows the source |
| Earn: counter | ticks up as each coin lands (shows a lagging value), chip pops `1.08×` | The number "fills up" |
| Earn: gain text | `+X` in `GAIN` gold with a dark outline, under the chip, rises a little, sums while more keeps coming, fades `1.2 s` after the last gain | Player knows exactly how much |
| Spend: loss text | `−X` in `LOSS` red with a dark outline, under the chip, **falls** a little, fades `1.0 s` | Clear cost, different direction and color |
| Spend in steps | paying something bit by bit (standing on a price tag) **sums** into one growing `−X` | One readable number, not a flicker |
| Spend: counter | drops immediately; chip shakes slightly (`±4` px, `0.15 s`) | Money is gone now |
| Applies to | **every** spend: upgrades, unlocks, shop items, floor price tags | Consistency |
| Automatic income | no flying coins; only `+X` and the counter tick | Not distracting while the game plays itself |
| Sound | coin tick per landing coin (pitch rising), `kaching` for big claims | Audio reward |

## How it works

### Where coins fly from

| Source | Coins fly from |
|---|---|
| Player picks up <currency> in the world | the player's screen position |
| A claim button (daily, goals, gift, welcome back, free <currency>) | that button's center |
| Automatic income (workers, idle, offline already shown) | no flight (quiet) |

```
 [🪙 1,250]          [🪙 1,330]              [🪙 1,200]
    ↑ 🪙 🪙             +80 (gold, rising)      −130 (red, falling; chip shakes)
  coins arcing in
```

## Rules

- The displayed number never exceeds the real balance; it only lags while coins are in the air.
- Gain and spend share one spot under the chip: a spend resets a running gain total and the other
  way round; the last direction wins.
- Big wins (≥ 10× a normal payment) add confetti.
- All amounts go through one formatter (`1,234`, `12.3K`, `4.56M`, rounded down).
- Spending can't go below zero; a failed purchase shakes the price button and plays the error sound.

## Implementation — Godot 4

The HUD listens to the save singleton's `money_changed(value)` signal and compares with the last
value: delta > 0 → fly coins (`UiKit.fly`) + gain text; delta < 0 → loss text + chip shake. Two hints
set before a change: `money_from(screen_pos)` (where the next gain's coins start) and `money_quiet()`
(the next gain is automatic). Keep `_shown` (lagging) and `_target` (real) values.

## Other engines

- Unity: an event `CurrencyChanged(delta, source)`, a pooled coin prefab and DOTween paths; the same
  gain/loss label logic.

## Done when

- [ ] Picking up <currency>, claiming a popup reward and buying something each show the right
      animation and text (capture mid-flight at real-time waits).
- [ ] Paying a price tag in steps shows one growing `−X`.
- [ ] Automatic income shows only `+X`.
