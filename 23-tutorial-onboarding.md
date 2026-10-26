# 23 — Tutorial and onboarding

> Teach the core loop in the first minutes by doing, not reading, and never repeat it.

## What it is

**Onboarding** is everything a new player experiences in their first session. The **tutorial** is
the guided part of it: a short sequence of steps where the game shows one action at a time
("move here", "tap this", "buy that") and waits until the player does it. Each step has a short
instruction, a pointer at the target, and a "done" condition checked against the game state.

Later systems (shop, daily rewards, staff, events) are not part of the first tutorial; they are
introduced **just in time**, the first time the player reaches them.

## Why it matters

- Most players decide in the **first 1–3 minutes** whether to keep a game. Day-1 retention is
  lost mostly in the first session: confused players leave silently.
- A player who completes the core loop once (earn → spend → see progress) is far more likely
  to return the next day.
- Walls of text get skipped; players learn by doing.
- A tutorial that repeats for returning players (or after an update) is a top review complaint.

## When to use it

- **Needed:** any game with a loop the player can't guess from the first screen (idle/tycoon,
  management, crafting, RPG, strategy).
- **Light version:** arcade, shooter, runner: one control drill ("drag to move", "tap to jump")
  and the first level does the rest.
- **Level-as-tutorial:** puzzle and match games: the first 3–5 levels are designed to teach one
  mechanic each, with a hand pointer on level 1.
- **Skip:** nothing to teach beyond the controls hint (`07`).

## When it happens

```
 first launch ─▶ splash (08) ─▶ consent (18) ─▶ tutorial starts
                                                  ├─ idle/tycoon: straight into play (no menu first)
                                                  └─ menu-first games: menu ▶ Play ▶ tutorial level
 tutorial done ─▶ normal play; new systems get a one-time intro when first reached
 returning player ─▶ never sees finished steps again
```

- Starts on the **first launch**, right after the splash and the consent form.
- For idle/tycoon games, drop the player **straight into play**; the main menu appears from the
  next launch.
- Ends after the player completes one full loop and makes their first purchase/upgrade.

## Before you build (for Claude)

1. Confirm the game needs a tutorial and which kind (full steps, control drill, or
   level-as-tutorial). Ask if unsure.
2. Show **Defaults** and ask: *"Use these values, or change any?"* — wait for the answer.
3. **Ask the user:**
   - The core loop as a list of actions (each becomes one step).
   - Which systems unlock later and should get a just-in-time intro.
   - Should the first launch skip the main menu?

## Defaults

| Setting | Default | Why |
|---|---|---|
| Style | **do, don't read**: one short instruction + a pointer at the target | Players learn by doing |
| Instruction | tip pill above the main button (`04`), ≤ 10 words | Readable at a glance |
| Pointer | guide arrow on the floor (3D) or hand pointer (2D UI) at the target | No hunting |
| Steps | one per core action (move, earn, collect, spend, first upgrade) | One thing at a time |
| Length | under `3` min | Before attention drops |
| First launch | `straight into play` (idle/tycoon), `menu first` (others) | Fastest path to fun |
| Hidden during tutorial | side buttons, next-unlock strip, special customers/events, failure states, interstitial ads, rate and notification asks | Less noise |
| Failure | none during the tutorial (customers don't leave, no game over) | No early frustration |
| Done | stored in the save; short celebration toast | Never repeats |
| Just-in-time intros | first time a system appears: reveal popup (`03`) + one pointer | Teach when relevant |

## How it works

- Each step: `{ text, target(), done() }`. Every frame, the tutorial skips forward over steps whose
  `done()` is already true, then shows the current step's text and points at `target()`.
- A target may be empty (e.g. "wait for the customer to finish"): show only the text.
- **Resume from saved progress**, never from session counters:
  - each `done()` reads saved state (unlocks owned, levels bought, currency earned), so a step
    finished last session stays finished;
  - if the save is clearly past the basics (first worker hired, first area unlocked), mark the
    tutorial done or jump to the remaining steps.
- **Returning players** after an update that adds a step: only the new step shows, never the old ones.
- **Off-screen targets**: an edge arrow points the way.

## Rules

- Never explain a system the player hasn't reached yet.
- Never block the screen with text the player must read before playing.
- Never show ads, rating or notification prompts during the tutorial.
- The tutorial can't soft-lock: if the player has no money for the next step, the step guides
  them to earn it.
- The first movement hint is "Drag to move" (`07`).
- Analytics: log each step reached (`20`) to find where new players quit.

## Implementation — Godot 4

- A `Tutorial` node with an array of steps (`text`, `target: Callable`, `done: Callable`).
- `_process`: advance while `done()` is true; set the HUD tip and the guide arrow; when all
  done, set `tutorial_done` in the save and stop processing.
- On setup, check saved progress and skip finished steps (e.g. staff hired → done).
- Systems check `GameState.tutorial_done` before showing extras.
- Tests: fresh save starts at step 1; a save with progress never shows finished steps.

## Other engines

Unity: a TutorialController with a list of ScriptableObject steps (predicate + target);
same resume logic.

## Done when

- [ ] A new player finishes in under 3 minutes without reading anything long.
- [ ] Closing the game mid-tutorial and reopening continues at the right step.
- [ ] A save with progress never sees finished steps (unit test).
- [ ] No ads, rating or notification prompt appears during the tutorial.
- [ ] Every later system has a one-time intro when first reached.
