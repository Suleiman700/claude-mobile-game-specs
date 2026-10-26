# 13 — Notifications (local reminders)

> A few friendly, useful reminders that bring players back, and a polite way to ask permission.

## What it is

Notifications scheduled **by the game on the device** (no server): "Your daily reward is ready",
"Your shop earned a lot while you were away", "We miss you". Plus an in-game "soft-ask" card that
explains why before the system permission prompt appears.

## Why it matters

- Reminders are one of the strongest ways to bring players back on day 1, 3 and 7.
- On Android 13+ and iOS, players must allow notifications; a cold system prompt on first launch is
  usually denied forever. Explaining first (soft-ask) gets far more "Allow".
- Too many or pushy notifications get the game muted or uninstalled.

## When to use it

- Games with reasons to come back: daily rewards, timers (energy, crops, production), offline
  earnings, events.
- Skip for games with no return hooks, and for games for young children unless the user insists.

## When it happens

- Soft-ask: at a calm moment (main menu or after a reward) from session 3.
- Scheduling: when the player leaves or pauses the game; everything is cancelled when they return.

## Before you build (for Claude)

1. Confirm the game has return hooks worth reminding (ask if unsure).
2. Show **Defaults** and ask: *"Use these values, or change any?"* Wait.
3. **Ask the user:**
   - the game's "come back" hooks (daily reward, energy full, production full, event),
   - reminder texts per language (offer drafts in the game's tone),
   - whether to deep-link a notification to a screen.
4. Local notifications need a plugin (a local-notification plugin for Android/iOS). **Review it
   with the user before installing.**

## Defaults

| Setting | Default | Why |
|---|---|---|
| Soft-ask card | from session `3`, at a calm moment: "Turn on reminders? We'll tell you when your daily reward is ready." **Yes** / **Not now** | Players understand why first |
| System permission | requested **only after Yes** (Android 13+ `POST_NOTIFICATIONS`, iOS authorization) | Avoids a permanent "Don't allow" |
| Re-ask | after `3` days if Not now | Gentle |
| Max soft-asks | `3` | Never nag |
| Ask budget | shares one ask per session with `12` and `19`; the soft-ask has first priority (`03`) | Never several asks at once |
| Settings switch | Notifications ON/OFF in Settings (`10`); turning it ON there asks the system directly | User control |
| Schedule after leaving | day `1` reward/missions, day `2` challenge, day `3` "we miss you", day `5` extra, day `7` "we miss you" | Proven comeback cadence |
| Time of day | `19:00` local | Evening play time |
| Quiet hours | nothing between `21:00` and `09:00` | Respectful |
| Daily cap | `1` per day | Never spammy |
| Event reminders | daily reward ready, production/offline earnings full, gift waiting | Useful, not advertising |
| On return | cancel all scheduled; re-plan on leave/pause | Always relevant |
| Android channels | `rewards`, `reminders` | Players can mute one kind |

## How it works

```
 session ≥ 3 and calm moment ─► soft-ask card
      ├─ Yes     ─► system permission prompt ─► allowed: schedule on leave
      └─ Not now ─► asks += 1, next ask in 3 days (max 3)

 player leaves ─► plan(): pick up to 1 per day, days 1/2/3/5/7 at 19:00, skip quiet hours
 player returns ─► cancel all
```

## Rules

- Texts: short, warm, in the player's language and the game's tone; no guilt-tripping, no fake
  urgency.
- Never promise a reward the game won't give when the player opens it.
- Tapping a notification opens the game (optionally deep-links to the right screen).
- Soft-ask state (asks, answer, next ask date) is stored in the settings/meta save.
- If the system permission is denied, never show the soft-ask again; the Settings switch explains
  how to enable it in system settings.

## Implementation — Godot 4

- A pure policy script: `should_soft_ask(sessions, state, has_permission)`, `after_answer(state, yes)`,
  `plan(hooks) -> Array` (unit-tested headless).
- A `Notify` wrapper talks to the plugin singleton (permission status/request, schedule, cancel all)
  and does nothing on desktop.
- Call `Notify.reschedule()` on `NOTIFICATION_APPLICATION_PAUSED` and `cancel_all()` on resume.

## Other engines

- Unity: Mobile Notifications package (Android channels, iOS authorization); the same policy class.

## Done when

- [ ] Policy tests pass: session gate, 3-day re-ask, max 3, no ask after a denial.
- [ ] On a device: soft-ask after session 3; Yes shows the system prompt.
- [ ] Reminders arrive at 19:00, at most one per day, never in quiet hours; all cancelled on return.
