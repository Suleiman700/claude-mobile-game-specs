# 17 — Testing and screenshot verification

> "It compiles" is not done: every feature is proven by a test and every screen by a real screenshot.

## What it is

Two habits used on every change:

1. **Headless tests**: scripts that run the game without a window and check logic, systems and UI
   flows, printing `N checks, M failed`.
2. **Screenshot captures**: a script that opens each screen in a real window at phone sizes, saves
   PNGs, and those images are reviewed against the design (alignment, sizes, clipping, contrast).

## Why it matters

Games break in ways players see instantly: a button off-screen on a tall phone, a popup that never
closes, a reward paid twice. Tests catch logic bugs; screenshots catch visual bugs that code review
never sees. Together they let Claude say "done" honestly.

## When to use it

- **Every game, every change.** Pure logic gets unit checks; systems get simulations; every UI change
  gets captures at two phone sizes.

## When it happens

After each feature or fix, before reporting it as done; and before every release (full run).

## Before you build (for Claude)

1. Confirm a test runner exists; if not, propose creating one.
2. Show **Defaults** and ask: *"Use these values, or change any?"* Wait for the answer.
3. **Ask the user:**
   - the target phone sizes (default below);
   - the engine binary path, if not obvious.

## Defaults

| Setting | Default | Why |
|---|---|---|
| Test runner | headless run of a test scene/script; exit code = number of failures | Works in CI and by hand |
| Saves in tests | a **throwaway** save path (e.g. `user://test_save.json`), deleted after | Never touch the player's save |
| Captures | a separate script that loads screens **as the real current scene** and saves PNGs | Real flows, not fakes |
| Waits | **real time** (timers), never frame counts | Windows can render hundreds of fps |
| Phone sizes | `405×720` (9:16) and `300×650` (9:19.5) | Normal and tall phones |
| Window size | must fit the monitor | Too-tall windows can render blank |
| Review | UI audit: sibling sizes, alignment, spacing, clipping, contrast, placement; use the `harsh-design-critic` skill on the captures when installed | Catches "almost right" |

## How it works

### What to test

| Kind | Examples |
|---|---|
| Pure logic | economy formulas, streak rules, ad/rating/notification policies, money formatting |
| Simulation | a bot plays the core loop for 2 minutes; workers do their jobs; enemies spawn |
| World geometry | spawn spots are open floor (physics shape query); walking routes don't cross walls (ray per leg) |
| UI flows | popups open/close, badges match, pause → main menu leaves no game behind |
| Dev tools | DEV buttons do what they say |

### What to capture (every UI change)

Main menu (fresh and returning), settings, game screen (tutorial, mid, late), every popup, each tab of
each sheet with every row state, money animations mid-flight (+X and −X), labels near screen edges.

### The harness-vs-game check

If a capture looks wrong, first check the capture script: frame-count waits, scenes added without
being the current scene, or a resized window can all produce fake bugs. Fix the harness before
"fixing" the game.

## Rules

- New script classes may need an import pass before tests run; run it twice if a theme references a
  newly added font.
- Report honestly: show failing checks; call partial fixes partial.
- Tests and captures never use the real save, settings, or ad IDs.
- Keep tests fast (under a few minutes) so they run on every change.

## Implementation — Godot 4

- `godot --headless --path . -s res://tests/run_tests.gd` → loads the test script at runtime (after
  autoloads), runs `check(condition, "name")` calls, prints the summary, exits with the failure count.
- Captures: a `SceneTree` script run with a window (`--resolution 405x720`), using
  `create_timer()` waits and `get_viewport().get_texture().get_image().save_png()`.
- `--import` pass before tests when new classes or assets were added.

## Other engines

Unity: Unity Test Framework (Edit Mode + Play Mode tests), screenshots via `ScreenCapture` in a
batch-mode player at fixed resolutions.

## Done when

- [ ] All tests pass.
- [ ] Captures reviewed at both phone sizes; audit findings fixed or listed.
- [ ] No test or capture touched the real save.
