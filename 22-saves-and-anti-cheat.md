# 22 — Saves and anti-cheat

> Keep the player's progress safe from crashes, bad updates and casual cheating.

## What it is

The save system stores everything the player has earned (currency, unlocks, levels, settings,
purchases) on the device and loads it on the next launch. Anti-cheat is the light protection on
top: it makes editing the save file, scanning memory or changing the phone's clock not worth
the effort.

## Why it matters

- Lost progress is the fastest way to get a 1-star review and an uninstall.
- A save that half-writes during a crash or battery death corrupts everything.
- Updates change the data shape; without migrations old players lose progress.
- Free currency cheats break the economy, ad revenue and in-app purchase value.

## When to use it

- **Every game with progress** (idle, tycoon, RPG, puzzle with levels, shooter with upgrades).
- Anti-cheat depth scales with money at stake: offline single-player games need the light layer
  below; games with leaderboards, trading or real-money items need server validation (out of scope).
- Skip only for pure arcade games with no persistent progress (and even then save settings).

## When it happens

- **Load:** at boot, before the main menu.
- **Save:** after every purchase or unlock, on a short autosave timer, and when the app is
  minimized, loses focus or closes.

## Before you build (for Claude)

1. Confirm what the game must remember (ask if unsure): currencies, unlocks, levels, settings,
   purchases, tutorial state, daily/retention state.
2. Show **Defaults** and ask: *"Use these values, or change any?"* — wait for the answer.
3. **Ask the user:**
   - Does the game need cloud save (Google Play Games Saved Games)?
   - Are there leaderboards or real-money items that need server checks?
   - Is there an existing save format that must be migrated?

## Defaults

| Setting | Default | Why |
|---|---|---|
| Format | `JSON` with a `version` number | Readable, easy to migrate |
| Files | one progress file per save slot/level; **settings in a separate file** | New game keeps settings |
| Write | **atomic**: write `save.tmp`, then rename over `save.json` | No half-written saves after a crash |
| Autosave | every `10 s` + after every purchase/unlock + on pause/minimize/close | Never lose more than seconds |
| Backup | keep the previous save as `save.bak` | Recover from a bad write |
| Migration | `version` → step-by-step migrate functions | Old saves always load |
| Integrity | HMAC seal of the content (key in code, obfuscated) | Hand-edited saves are detected |
| On tamper | fall back to the last good backup | No reward for editing |
| Memory | currencies held scrambled (XOR with a per-run key) | Blocks memory scanners |
| Time cheats | rewards use the stored last-seen time; clock going backwards grants nothing until it catches up; offline time capped | Daily/offline rewards can't be farmed |
| Speed hacks | compare game time with the audio/monotonic clock; if too fast, scale income down | Speed tools stop paying |
| Cloud save | `off` (on: Play Games Saved Games, merge by most progress) | Optional |

## How it works

```
 launch ─▶ load save.json ──ok──▶ verify HMAC ──ok──▶ migrate(version) ─▶ game
                │ fail                  │ fail
                ▼                       ▼
           load save.bak ───────▶ (same checks) ─▶ or start fresh + keep settings
 any change ─▶ mark dirty ─▶ (purchase | 10 s timer | pause) ─▶ write tmp ─▶ rename ─▶ copy old to .bak
```

- **New game** wipes progress only; settings and purchases (remove-ads, starter packs) stay.
- **Migration example:** version 2 split a shared "staff speed" upgrade into per-role upgrades;
  the migrate function copies the old level into each new one so nobody loses value.

## Rules

- Tests and screenshot tools always use a **throwaway save path**; they never touch the
  player's real save.
- Never trust the client for purchases; restore them from the store (`19`).
- Never delete a save the game can't read: rename it aside and keep it for support.
- Save on the main thread only after gameplay changes settle (no saving every frame).
- Keep the save small (< 1 MB); never store textures or derived data.

## Implementation — Godot 4

- A `GameState` autoload: `save_game()`, `load_game()`, `reset()`, signals for changes.
- A `SaveSeal` helper: HMAC-SHA256 of the JSON, atomic write (`FileAccess` to `.tmp`,
  `DirAccess.rename_absolute`), backup handling.
- A `Vault` helper for scrambled in-memory currency; a `Guard` autoload for the speed check.
- Save on `NOTIFICATION_WM_CLOSE_REQUEST` and `NOTIFICATION_APPLICATION_PAUSED`.

## Other engines

Unity: `File.Replace` for atomic writes, `HMACSHA256`, an obscured-types package for memory,
`OnApplicationPause(true)` to save.

## Done when

- [ ] Killing the app during a save never corrupts it (test by forcing a crash mid-write).
- [ ] Editing the JSON by hand is detected and the backup loads.
- [ ] Setting the clock back does not re-grant the daily or offline reward.
- [ ] An old-version save migrates with nothing lost (unit test per migration).
- [ ] New game keeps settings and purchases.
- [ ] Tests never touch the real save.
