# 20 — Analytics

> Measure where players quit and what they enjoy, with consent and without personal data.

## What it is

A small set of named events the game sends to an analytics service (typically Firebase Analytics):
tutorial steps, unlocks, sessions, ads watched, purchases, rating and notification answers. The
dashboards turn them into funnels and retention charts.

## Why it matters

Without data you guess. With a handful of events you learn that 40% of players quit at tutorial step 3,
that nobody claims goal #2, or that the "Double it" offer on gifts is the most-watched ad. That turns
into concrete fixes that raise retention and revenue.

## When to use it

- Any game meant to be published and improved after launch.
- Skip for prototypes, or when the user doesn't want any tracking (feature switch off, `25`).
- Always gated by consent (`18`).

## When it happens

Events fire at the moments listed below, during normal play; nothing is sent before consent allows it.

## Before you build (for Claude)

1. Confirm the game should collect analytics; ask if unsure.
2. Show **Defaults** and ask: *"Use these values, or change any?"* Wait for the answer.
3. **Ask the user:**
   - the analytics service (default Firebase, shared with push `14`);
   - extra game-specific events (e.g. `level_fail` with `level` for puzzle/runner games).
   The analytics SDK/plugin is third-party code: Claude reviews it with the user before installing.

## Defaults

| Event | Params | Why |
|---|---|---|
| `tutorial_step` | `step` | Where new players drop off |
| `tutorial_done` | `seconds` | Onboarding length |
| `level_start` / `level_end` | `level`, `result` | Difficulty curve (level-based games) |
| `unlock` | `id`, `minutes_played` | Progress pace |
| `upgrade` | `id`, `level` | What players value |
| `session_start` / `session_end` | `seconds` | Session length, retention |
| `ad_rewarded` | `placement` (double_gift, free_cash, welcome_back, revive) | Which offers work |
| `ad_interstitial` | `reason` (purchases, idle_catchup, level_end) | Ad pressure |
| `daily_claim` | `day` | Streak health |
| `goal_claim` | `id` | Goal design |
| `purchase` | `product` | Revenue sources |
| `rate_prompt` | `answer` (yes, no, later) | Rating funnel |
| `notify_soft_ask` | `answer` | Permission funnel |

| Setting | Default | Why |
|---|---|---|
| Before consent | send nothing | Privacy law and store rules |
| Debug builds | log to console (or a debug view), don't pollute live data | Clean dashboards |
| Naming | `snake_case`, ≤ 40 chars, ≤ 25 params per event | Service limits |

## How it works

Game code calls one wrapper (`Analytics.log(event, params)`) at the moments above. The wrapper
checks consent and the feature switch, then forwards to the SDK or drops the event.

## Crash reporting

Crash reports (e.g. Firebase Crashlytics) catch crashes players never report. Default: on in release builds once
Firebase is set up (switch `crash_reports`, `25`); collection follows the consent rules in `18`; never attach names,
emails or save contents; debug builds report to a separate test app so live data stays clean.

## Rules

- No personal data in events (no names, emails, device IDs in params).
- Event names stay stable forever; renaming breaks history.
- Track decisions you'll actually act on; don't log everything.
- One wrapper only; game code never calls the SDK directly.

## Implementation — Godot 4

- An `Analytics` autoload: `log(event: String, params := {})`, `set_enabled(bool)` called from the
  consent flow, no-op when the plugin is absent (desktop, tests).
- Calls placed in the tutorial, unlock/upgrade code, ads callbacks, store callbacks, rate and
  notification prompts.

## Other engines

Unity: Firebase Unity SDK `FirebaseAnalytics.LogEvent`, same wrapper class and event list.

## Done when

- [ ] A test device shows the tutorial funnel and ad events in the debug view.
- [ ] Nothing is sent before consent; denying consent stops all events.
- [ ] Event names match the table.
