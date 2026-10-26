# 12 — Rate prompt

> Ask happy players for a store rating, inside the game, at a good moment, rarely.

## What it is

A two-step request for a rating: first a small in-game question ("Enjoying <Game>?"), then — only
for players who say yes — the store's own in-app review card (Google Play In-App Review, iOS
StoreKit review), which lets them rate without leaving the game.

## Why it matters

- Store rating and review count strongly affect downloads and ranking.
- Asking at the wrong moment (after a loss, during an ad) produces bad ratings.
- The pre-question sends unhappy players to a feedback form instead of a one-star review.

## When to use it

- Every game published on a store with ratings.
- Skip in builds outside the stores (web, sideloaded) or replace with a store link.
- Games for young children: usually skip (store policies on prompting kids).

## When it happens

After the player has formed an opinion (session 3 or later), right after a **good moment**: a level
completed, a big reward claimed, a new unlock, a new rank/star. Never after a failure.

## Before you build (for Claude)

1. Confirm the game is (or will be) on a store with ratings (ask if unsure).
2. Show **Defaults** and ask: *"Use these values, or change any?"* Wait.
3. **Ask the user:**
   - the feedback email address for unhappy players,
   - which moments in this game count as "good moments",
   - the game's store page URL (fallback when in-app review isn't available).
4. In-app review needs a plugin (an in-app review plugin wrapping Google's ReviewManager on Android,
   StoreKit on iOS). **Review it with the user before installing.**

## Defaults

| Setting | Default | Why |
|---|---|---|
| Earliest | after session `3` | Players have formed an opinion |
| Trigger | right after a **good moment** (level complete, big claim, new unlock/star) | Happy players rate higher |
| Never | after a loss/fail, after an ad, during the tutorial, while other popups are queued | Bad timing → bad ratings |
| Step 1 | popup "Enjoying `<Game>`?" → **Yes!** (green) / **Not really** (neutral) | Routes unhappy players away from the store |
| Yes | open the store's **in-app review** card | Highest completion, no leaving the game |
| Not really | "Tell us what to improve" → opens a feedback email prefilled with game + version | Feedback instead of a bad review |
| Later / ✕ | ask again after `7` days | Gentle |
| Max asks | `3` in total; never again after Yes or Not really | Never nag |
| Ask budget | shares one ask per session with `13` and `19` (`03`, Ask budget) | Never several asks at once |
| Fallback | if in-app review isn't available, open the store page | Still works off Play services |
| Reward for rating | **never** | Store policies forbid rewarding ratings |

## How it works

```
 good moment ─► due()? ─► "Enjoying <Game>?"
                              ├─ Yes!       ─► in-app review card (store decides if it shows) ─► done forever
                              ├─ Not really ─► feedback email ─► done forever
                              └─ ✕ / Later  ─► asks += 1, next ask in 7 days (max 3)
```

- `due()` checks: session ≥ 3, not answered, asks < 3, 7 days since the last ask, no fail just
  happened, no popup chain running, the session's ask slot is free (`03`).
- The store limits how often its card really appears; the game can't know if the player rated, so
  after **Yes** it marks the prompt as done.

## Rules

- Short, friendly copy; never "Give us 5 stars".
- Store prompt state (sessions, asks, last ask date, answered) in the settings/meta save, not the
  game save, so "New game" doesn't ask again.
- Never block play: the prompt waits for the current reward/level-complete popup to close.

## Implementation — Godot 4

- A pure `RatePrompt.due(state, today) -> bool` (unit-tested) and `RatePrompt.ask_then(callable)`
  hooked into the "good moment" flows.
- Android: the plugin's singleton `requestReview()`; desktop/debug shows the pre-question and logs
  instead of calling the store.

## Other engines

- Unity: Google Play In-App Review package (`ReviewManager`), iOS `SKStoreReviewController`; the same
  `due()` policy.

## Done when

- [ ] Unit tests for `due()`: session gate, 7-day retry, max 3 asks, never after a fail.
- [ ] Debug flow shows the pre-question; Yes calls the review API; Not really opens the email.
- [ ] No reward is ever given for rating.
