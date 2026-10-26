# 25 — Feature switches

> Turn whole systems on or off per game or per build without changing code.

## What it is

Feature switches are simple on/off settings for optional systems (daily rewards, ads, push,
special customers...). Code checks the switch before showing the system's UI or running its logic.

## Why it matters

- One codebase serves many games: each game picks the systems it wants.
- Systems that need setup (Firebase, store products) stay off until they're ready.
- A broken system can be turned off for a release without ripping out code.
- Testing each system on and off catches hidden dependencies.

## When to use it

- Any game built from reusable systems, or any optional system that needs outside setup.
- Skip for tiny one-off prototypes.

## Before you build (for Claude)

1. Confirm which optional systems this game ships with. Ask if unsure.
2. Show **Defaults** and ask: *"Use these values, or change any?"* — wait for the answer.
3. **Ask the user:**
   - Which systems are ready now and which need outside setup first?
   - Should any switch be remote (changeable without an update)?

## Defaults

| Switch | Default | Why |
|---|---|---|
| `daily_rewards` | on | Core retention |
| `goals` | on | Core retention |
| `gifts` | on | Light reward |
| `special_customers` (VIP) | on | Variety |
| `offline_earnings` | on | Idle games expect it |
| `ads` | on | Revenue |
| `iap` | `off` until store products exist | Needs setup |
| `notifications` | on | Retention |
| `push` | `off` until Firebase is set up | Needs setup |
| `rate_prompt` | on | Ratings |
| `analytics` | `off` until Firebase is set up | Needs setup |
| `crash_reports` | `off` until Firebase is set up | Needs setup |
| `dev_panel` | debug builds only, not switchable in release | Never ship it |

## How it works

- Each switch is a project setting (bool). A `Features.on(id)` helper reads it.
- Off means **completely absent**: no buttons, no badges, no timers, no popups, no network calls.
- Systems that depend on another one check both (e.g. "double it" buttons need `ads`).

## Rules

- Every new optional system ships behind a switch.
- Turning a switch off must never leave a dead button or an empty slot in the layout.
- Tests run with all switches on and with each one off.

## Implementation — Godot 4

Project Settings `<game>/features/<id>` (bool) with defaults; `Features.on(id)` reads
`ProjectSettings.get_setting()`; UI and systems guard with it.

## Other engines

Unity: a `FeatureFlags` ScriptableObject, or Remote Config with local defaults.

## Done when

- [ ] Each switch turned off removes its UI and logic with no errors.
- [ ] Layouts still look right with any combination off.
- [ ] The dev panel is absent from release builds regardless of switches.
