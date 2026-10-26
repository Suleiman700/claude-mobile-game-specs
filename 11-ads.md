# 11 — Ads

> Banner, rewarded and full-screen ads placed so they earn money without driving players away.

## What it is

Three ad formats from an ad network (usually Google AdMob):

- **Banner**: a thin strip at the bottom of the screen, always visible.
- **Rewarded video**: the player *chooses* to watch a video for a reward (double a gift, free coins).
- **Interstitial**: a full-screen ad shown at a natural pause.

## Why it matters

- For free games, ads are usually the main income.
- Badly placed ads (mid-action, too often, fake close buttons) are the top reason players leave
  one-star reviews and uninstall.
- Rewarded ads are liked by players when they are optional and generous, and they earn the most.

## When to use it

- Free-to-play mobile games: idle, tycoon, casual, puzzle, arcade, hyper-casual.
- Skip for paid games, premium games, and be extremely careful with games for children (see `18`:
  special rules, often no personalized ads at all).
- Rewarded ads need something worth rewarding (currency, boosts, extra lives).

## When it happens

- Banner: from the main menu on, all the time.
- Interstitial: right after a purchase/upgrade milestone, or as an idle catch-up at the next reward
  interaction (see Defaults).
- Rewarded: only when the player taps a button with the ▶ video icon.

## Before you build (for Claude)

1. Confirm the game should have ads and which formats (ask if unsure).
2. Show **Defaults** and ask: *"Use these values, or change any?"* Wait.
3. **Ask the user:**
   - AdMob App ID and ad unit IDs (stored in an ignored config file, never in code),
   - whether the game has a remove-ads purchase (`19`),
   - whether the game targets children (`18`),
   - the rewarded placements this game offers.
4. Ads need a third-party plugin (for Godot e.g. Poing Studios' AdMob plugin, MIT; core only, no
   mediation adapters unless asked). **Review it with the user before installing** (source,
   license, what it downloads) and follow any package-approval rules.

## Defaults

| Setting | Default | Why |
|---|---|---|
| Banner | bottom, **always on** (menu and game); UI sits above it | Steady income, predictable layout |
| Interstitial | after every **`5` purchases or upgrades** (unlocks and upgrade levels count) | A natural pause right after spending |
| Idle catch-up | if **no purchase/upgrade for `15–20` min** of play, show one interstitial on the **next reward/collect interaction** (collecting money, claiming a reward), then reset the timer | Long sessions without buying still see some ads, at a happy moment, never mid-action |
| Time-based interstitials | **none** (never "every X minutes") | Timers interrupt play at random moments |
| Never show an interstitial | first `2` min of the first session, during the tutorial, mid-action or mid-dialog, right after a rewarded ad | Not annoying |
| Interstitial delay | `0.6 s` after the purchase animation | The purchase feels complete first |
| Rewarded placements | "Double it" on gifts/rewards, 2× offline earnings, level-complete bonus, a "free currency" button with a `30` min cooldown | Optional, generous |
| Rewarded fallback | if the ad fails or is closed early, pay the **base** reward (never nothing) | Fairness |
| Remove-ads purchase | removes banner + interstitials; rewarded stays available | Standard and policy-friendly |
| Content rating cap | `G` | General audience |
| Debug builds | the network's **test ad IDs** always | Real ads during development can get accounts banned |
| Desktop debug | **mock ads**: full-screen "TEST AD" overlay with a 3 s countdown, grey "TEST BANNER" bar | Every flow testable on a computer |
| Automated tests | no ads at all | Deterministic tests |

## How it works

```
 purchase/upgrade ──► count += 1 ──► count % 5 == 0 ? ──► wait 0.6 s ──► interstitial
 no purchase for 15–20 min ──► flag "catch-up due" ──► next collect/claim ──► interstitial ──► reset
 tap "▶ Double it" ──► pause game ──► rewarded video ──► closes ──► pay ×2 (or ×1 if it failed)
```

- The game pauses while any full-screen ad is showing; rewards are paid **after** it closes.
- Ads load early and retry with backoff (`5 → 10 → 20 → 40 → 60 s`); reload when the app resumes.
- Banner height is measured; layouts move up by it and react when it appears or disappears.
- Counters (purchases since the last interstitial, time of the last purchase) are saved.

## Rules

- Rewarded buttons always show the ▶ video icon and the "ad" color (`02`), and hide when no ad is
  loaded.
- No fake close buttons, no ads where a button was a moment ago, no ads during input.
- Consent first (`18`): no ad request before consent is resolved.
- Never more than one full-screen ad in a row; never an interstitial right after a rewarded one.

## Implementation — Godot 4

- An `Ads` autoload with three backends: the real plugin on Android, `mock` on desktop debug
  builds, `none` in headless tests.
- API: `show_banner()`, `hide_banner()`, `banner_height()`, `is_rewarded_ready()`,
  `show_rewarded(on_reward, on_closed)`, `show_interstitial(then)`, `video_icon()`.
- Look up plugin classes by global class name so the project still runs without the plugin.
- IDs from `config/ads.cfg` (git-ignored) with a committed `config/ads.example.cfg`.
- The game calls `after_purchase()` (cadence counter) and `on_reward_interaction()` (catch-up check).

## Other engines

- Unity: Google Mobile Ads Unity plugin; the same backend switch (Editor = mock) and the same cadence
  rules in an `AdPolicy` class.

## Done when

- [ ] Mock flow: 5 purchases → one interstitial; 15+ min with no purchase, then a collect → one interstitial.
- [ ] "Double it" pays double after the mock ad, and the base amount when the ad is unavailable or fails.
- [ ] No interstitial in the first 2 minutes, during the tutorial, or right after a rewarded ad.
- [ ] Banner never covers buttons at 9:16 and 9:19.5.
- [ ] Debug builds use test IDs; real IDs live only in the ignored config.
