# 15 — Retention systems

> Small, friendly reasons to come back tomorrow: daily reward, daily goals, gifts, offline earnings
> and special customers.

## What it is

A set of optional systems that reward players for returning and for playing a little every day:

- **Daily reward**: a 7-day login streak with growing rewards.
- **Daily goals**: 3 small tasks per day with a reward each.
- **Gifts**: a surprise box that appears in the world now and then.
- **Offline earnings**: money earned while the game was closed ("Welcome back").
- **Special customers (VIP)**: rare visitors who pay much more.

## Why it matters

Most mobile games lose the majority of players in the first days. These systems give a concrete
reason to open the game again ("my day 7 reward is ready", "my staff earned money while I was away"),
create small goals between big ones, and add surprise. They also create natural, optional places
for rewarded ads ("Double it").

## When to use it

| System | Good fit | Skip or adapt |
|---|---|---|
| Daily reward | almost every free-to-play game | premium/paid games without a currency |
| Daily goals | idle, tycoon, restaurant, shooter, runner, puzzle | very short story games |
| Gifts in the world | games with a walkable world (tycoon, restaurant, idle) | puzzle/runner: use a gift button or a gift between levels instead |
| Offline earnings | idle/tycoon games with automation (workers, machines) | games without passive income |
| Special customers | games with customers/clients (shop, restaurant, hotel) | shooter: use "golden enemy"; runner: "bonus coin streak" |

Each system is a separate feature switch (`25`), so a game can ship only the ones that fit.

## When it happens

```
App opens → (consent) → Welcome back (offline earnings) → Daily reward → play
During play → goals progress, gifts drop every 15–30 min, special customers appear
Player leaves → offline timer starts, notifications planned (13)
```

Startup popups run one after another, never stacked.

## Before you build (for Claude)

1. Confirm which systems this game needs (table above); ask if unsure.
2. Show **Defaults** and ask: *"Use these values, or change any?"* Wait for the answer.
3. **Ask the user:**
   - what one "normal payment" is in this game (one customer, one level, one kill wave), so
     rewards scale with progress;
   - the 3 daily goal types that fit the game;
   - what starts offline earnings (first worker, first machine);
   - the game's "special customer" equivalent, if any.

## Defaults

Rewards are counted in **normal payments** (`N`), so they stay meaningful as the game grows.

| Setting | Default | Why |
|---|---|---|
| Daily reward days | `7` tiles: `3, 4, 5, 6, 8, 10, 25` × N | Growing rewards, day 7 is the big one |
| Streak | consecutive days walk 1→7 and loop; **missing a day restarts at day 1** | Clear rule, strong habit |
| Daily reward popup | on start (after Welcome back), must be claimed; also from a Daily button (closable) | Seen every day |
| Daily goals | `3` per day, reset at local midnight | Small, doable targets |
| Goal reward | `5–8` × N each | Worth doing |
| Goal buttons | **GO** (grey, closes the popup) → **CLAIM** (green, pulsing) → **DONE** | Obvious state |
| Gift frequency | one every `15–30` min of play | Rare enough to feel special |
| Gift spot | random **reachable, open** spot (not inside walls/furniture, not on a buy pad) | Player can always get it |
| Gift value | `3–5` × N | Nice but not game-breaking |
| Gift popup | **Double it ▶ ad** / **Nice!** | Optional ad |
| Offline earnings | `25%` of full-speed income, capped at `2 h` | Rewards returning, not quitting |
| Welcome back popup | "Your staff earned $X" → **Collect 2× ▶ ad** / **Collect** | Optional ad |
| Special customer chance | `1 in 12` after the tutorial | Rare surprise |
| Special customer pay | ×2 (70%) or ×3 (30%) | Exciting spikes |
| Special customer look | dressed up, gold crown + "VIP ×3" above the head | Easy to spot |

## How it works

### Daily reward

```
 ┌────────── DAILY REWARD ──────────┐
 │ Come back every day. Miss a day   │
 │ and it starts over.               │
 │ [Day1✓][Day2✓][Day3✓][Day4 ●]     │  ← claimed: check, faded · today: highlighted, pulsing
 │ [Day5 ][Day6 ][   Day7 🎁   ]     │  ← day 7: gold tile, gift icon
 │ [ ✓          Claim           ]    │
 └───────────────────────────────────┘
```

Claiming sends coins from the Claim button to the money counter (`05`); confetti on day 7. The Daily
button shows a `!` badge while today's reward is unclaimed.

### Daily goals

Each row: icon, title, progress bar `x/target`, reward. Examples by genre:

| Genre | Goal examples |
|---|---|
| Tycoon / idle | buy 2 upgrades, earn $500, unlock 1 area |
| Restaurant / shop | serve 15 customers, clean 10 tables, serve a VIP |
| Shooter | defeat 50 enemies, finish 3 waves, use a power-up 5 times |
| Runner | run 2,000 m, collect 300 coins, jump 50 times |
| Puzzle | win 3 levels, use 2 boosters, clear 100 tiles |

Footer: "New goals in 5h 12m". The Goals button badge shows how many are ready to claim.

### Gifts

A wrapped box drops onto an open floor spot (or appears as a button in games without a world).
Walking onto it or tapping it opens the gift popup.

### Offline earnings

On launch, time away × income rate × 25%, capped at 2 hours. Only once automation exists. The
Welcome back popup appears before the daily reward.

### Special customers

A rare visitor with a crown pays ×2 or ×3. Happy faces above them, a "VIP ×3 +$48" popup when they pay.

## Rules

- Retention rewards never block core play. Everything is claimable **without** ads; ads only double.
- Startup popup order: Welcome back → Daily reward → reveal popups. One at a time.
- Day boundaries use the device's local date; protect against clock tricks (`22`).
- Every system has DEV-panel shortcuts (`16`) and unit tests.
- Don't show any of these during the tutorial (`23`).

## Implementation — Godot 4

- Pure functions in the save autoload (e.g. `GameState`):
  `daily_index(last_day, last_date, today) -> int`, `claim_daily()`, `add_goal_progress(id, n)`,
  `claim_goal(goal)`, `collect_offline(pay: bool) -> int`. Easy to unit-test headless.
- HUD functions: `show_daily(from_button)`, `show_goals()`, `show_welcome_back(amount)`,
  `reward_popup(title, amount)`.
- Gift node: drops with a bounce, spins, opens the popup when the player stands on it.
- Spawn spots are a list of points checked by a physics shape query in tests.

## Other engines

Same pure functions in a `Progress`/`Meta` service class; UI as prefabs; tests with the engine's
test framework.

## Done when

- [ ] Unit tests: streak continues, loops after day 7, restarts after a missed day.
- [ ] Goals reset each day and pay once; badges match.
- [ ] Gift spots are proven reachable and clear (physics test).
- [ ] Offline earnings are capped and paid once per launch.
- [ ] Startup popups never overlap.
