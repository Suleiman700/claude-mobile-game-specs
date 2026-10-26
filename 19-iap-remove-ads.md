# 19 — In-app purchases and Remove Ads

> Let players pay to remove ads or buy currency, safely, fairly and restorably.

## What it is

Store purchases inside the game through Google Play Billing (and the App Store on iOS):

- **Remove Ads**: a one-time purchase that removes banners and full-screen ads.
- **Starter pack**: a one-time bundle (Remove Ads + currency) offered to engaged players.
- **Currency packs**: repeatable purchases of coins/gems.

## Why it matters

Many players dislike ads and will gladly pay a small amount to remove them; a few players spend on
currency. Purchases are often a large share of a casual game's revenue, and "Remove Ads" also
improves reviews. Done wrong (lost purchases, no restore), it causes refunds and 1-star reviews.

## When to use it

- Free games with ads: **Remove Ads** at minimum.
- Games with a currency economy (idle, tycoon, shooter upgrades, puzzle boosters): currency packs.
- Skip for premium (paid) games, or when the user doesn't want purchases yet (feature switch off, `25`).

## When it happens

- A **Shop** button (right-side HUD column) and a **+** on currency counters open the shop anytime.
- The starter pack is offered once after the player's 3rd session.
- Restore runs automatically on launch.

## Before you build (for Claude)

1. Confirm the game should sell anything; ask if unsure.
2. Show **Defaults** and ask: *"Use these values, or change any?"* Wait for the answer.
3. **Ask the user:**
   - product IDs and prices (the user creates them in the store console);
   - which packs to sell and what they contain;
   - license-tester accounts for testing.
   A billing plugin is third-party code: Claude reviews it with the user (source, license, what it
   downloads) before installing.

## Defaults

| Product | Type | Default |
|---|---|---|
| `remove_ads` | non-consumable | removes banners + full-screen ads; **rewarded ads stay** (player's choice) |
| Starter pack | non-consumable, one-time | Remove Ads + a currency bundle, offered after session `3` |
| Currency packs | consumable | `3` tiers (small / medium / big), "Best value" tag on one |

| Setting | Default | Why |
|---|---|---|
| Restore | automatic on launch + a Restore button in Settings (required on iOS) | Purchases never lost |
| Grant | only after the store confirms; then acknowledge/consume | No free items, no double charges |
| Pending purchases | show "Purchase pending", grant when complete | Some payment methods are slow |
| Starter pack offer | once, from session `3`, within the ask budget (`03`) | Never on top of other asks |
| Prices | the store's localized price string, never hard-coded | Correct currency everywhere |
| Remove Ads effect | banner hides **immediately**, no restart | Feels instant |

## How it works

```
Tap Shop → products queried → tap price → store sheet → success
  → grant item → acknowledge/consume → save → UI updates (banner hides for remove_ads)
Launch → query owned purchases → re-grant non-consumables (restore)
```

## Rules

- No randomized paid rewards for children; no fake countdown "sales".
- Purchase state is stored in the protected save (`22`) **and** restored from the store.
- Never trust client data alone for purchases; the store is the source of truth.
- Test only with license-tester accounts; never ship test product IDs.
- Rewarded ads remain available after Remove Ads; they are optional bonuses.

## Implementation — Godot 4

- A `Store` autoload wrapping the Google Play Billing plugin: `query_products()`, `buy(id)`,
  `restore()`, signal `changed`.
- The ads autoload checks a `premium` flag before showing banners or full-screen ads, and hides the
  banner when `changed` fires.
- No-op backend on desktop/web so the game runs everywhere.

## Other engines

Unity: Unity IAP (scripted or codeless) with the same product IDs; iOS uses StoreKit through it.

## Done when

- [ ] A license-tester purchase of `remove_ads` hides the banner instantly.
- [ ] Reinstall + launch restores Remove Ads.
- [ ] Consumable packs grant exactly once, even if the app is killed mid-purchase.
- [ ] Prices show in the local currency.
