# 14 — Push notifications and topics (Firebase)

> Server-sent messages to groups of players: news, events, comeback offers.

## What it is

Push notifications sent **from a server** (Firebase Cloud Messaging) instead of scheduled on the
device. Players are grouped by **topics** ("everyone", "Arabic speakers", "finished all content",
"paying players"), so one message reaches exactly the right group.

## Why it matters

- Announce updates, events and new content to everyone at once, without shipping a new build.
- Re-engage specific groups (players stuck at a level, players who finished everything).
- Topics avoid storing personal data: the server never needs a list of users.

## When to use it

- Live games with updates, events or seasons; games with a team to write messages.
- Skip for small single-player games with no live content: local reminders (`13`) are enough.

## When it happens

- Topic sync: on the first start of each app process (subscribe to the current groups).
- Messages: whenever the team sends them from the Firebase console or a server.

## Before you build (for Claude)

1. Confirm the game needs server pushes, not just local reminders (ask if unsure).
2. Show **Defaults** and ask: *"Use these values, or change any?"* Wait.
3. **Ask the user:**
   - the Firebase project (they add `google-services.json` themselves; it's config, not a secret,
     but keep it out of public repos if they prefer),
   - which topic groups this game needs and its progress buckets,
   - whether pushes may carry rewards or open screens.
4. Push needs a plugin (a Firebase Cloud Messaging plugin for the engine). **Review it with the user
   before installing.** Analytics from the same SDK follows consent (`18`, `20`).

## Defaults

| Topic | Meaning | Kind |
|---|---|---|
| `all` | everyone | always subscribed |
| `lang_<code>` | `lang_en`, `lang_ar`, `lang_es`… | one value at a time |
| `level_<bucket>` | progress buckets, e.g. `1_4`, `5_10`, `11_20`, `21_plus` | one value at a time |
| `content_<state>` | `playing` / `done` (finished all content) | one value at a time |
| `premium_<state>` | `free` / `paid` | one value at a time |
| `ver_<version>` | app version (unsubscribe the old one after an update) | one value at a time |
| `install_<yyyymm>` | install month cohort | set once |
| `opt_offers` | player agreed to receive offers | opt-in |
| `qa` | testers | opt-in |
| debug prefix | `dbg_` added to every topic in debug builds | Test pushes never reach players |

| Setting | Default | Why |
|---|---|---|
| Sync | first start of each process: subscribe current values, unsubscribe the other values of each one-value group | Groups stay correct |
| Data payload | optional `reward` (known ids like `coins_100`, paid once per message id) and `screen` (`shop`, `missions`, `settings`) | Rewards and deep links |
| App open (foreground) | show the message as an in-game toast, not a system notification | Not intrusive |
| Permission | shares the notification soft-ask (`13`) | One ask only |

## How it works

```
 app start ─► build topic set from state (language, progress, version, purchases)
          ─► subscribe new values, unsubscribe stale ones (dbg_ prefix in debug)
 console sends to "lang_ar" ─► device shows notification ─► tap ─► opens game (screen=shop)
                                                                   └─ reward id? pay once
```

## Rules

- Never put personal data in topics; topics are groups, not users.
- Rewards from pushes are validated: known ids only, once per message id.
- Write and translate each message per `lang_` topic on the console side.
- Respect quiet hours and frequency limits from `13` when scheduling campaigns.

## Implementation — Godot 4

- A pure `Push.topics(state) -> Dictionary` (topic → subscribed) with unit tests.
- `Push.sync()` calls the plugin singleton to subscribe/unsubscribe; no-op on desktop.
- A handler for opened messages reads `screen` and `reward` from the data payload.

## Other engines

- Unity: Firebase Unity SDK Messaging (`SubscribeAsync` / `UnsubscribeAsync`), the same topic function.

## Done when

- [ ] Unit tests for `topics()` cover each group and the version change.
- [ ] A debug device shows `dbg_*` topics in the Firebase console.
- [ ] A test push with `screen=shop` opens the shop; a reward id pays once only.
