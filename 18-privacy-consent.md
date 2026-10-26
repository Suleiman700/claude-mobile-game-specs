# 18 — Privacy and consent

> Ask for consent the right way before any ad, analytics or tracking SDK touches player data.

## What it is

The flow that asks players (where the law requires it, e.g. EEA, UK, Switzerland) whether ads may be
personalized and data may be used, using Google's **User Messaging Platform (UMP)** form, plus a
"Privacy options" button so players can change their answer later, and a privacy policy link.

## Why it matters

Ads, analytics and push SDKs process personal data. Without proper consent, ad networks can stop
serving ads, the store can reject or remove the game, and the developer can face legal trouble.
Getting it right first also protects ad revenue: properly consented traffic earns more.

## When to use it

- **Any game with ads, analytics, push or crash reporting.**
- Games directed at children need a stricter setup (child-directed treatment, no personalized ads);
  confirm with the user, never decide alone.
- Skip only for games with no third-party SDKs at all.

## When it happens

```
App start → splash → consent check (form shows only where required) → ads + analytics start → menu
Later: Settings → "Privacy options" (only when required) → player changes their choice
```

## Before you build (for Claude)

1. Confirm which SDKs the game uses (ads, analytics, push, crash reporting); ask if unsure.
2. Show **Defaults** and ask: *"Use these values, or change any?"* Wait for the answer.
3. **Ask the user:**
   - Is the game directed at children (under 13)? (a legal answer: the user decides)
   - Privacy policy URL.
   - Any country-specific needs (e.g. US state privacy laws).
   Write `TODO(confirm)` for anything legal that isn't answered.

## Defaults

| Setting | Default | Why |
|---|---|---|
| Consent tool | Google **UMP** (ships with the ads SDK) | Required by Google for EEA/UK ads |
| Order | update consent info → show the form if required → **then** start ads and analytics | No data before consent |
| Timeout | if nothing answers in `10 s`, start without consent (non-personalized) | Never blocks the game |
| Privacy options | Settings button, shown **only** when Google says it's required | Players can change their mind |
| Audience | general audience (not child-directed) unless the user says otherwise | Most casual games |
| Max ad content rating | `G` | Safe for everyone |
| Analytics | off until consent allows it | Respect the choice |
| Personal data | none collected by the game itself; SDKs declared in the store's data forms | Smaller legal surface |
| Privacy policy | link in Settings and in the store listing | Required by stores |

## How it works

1. On launch, request a consent info update.
2. If a form is available and consent is required, load and show it.
3. When it closes (or on timeout/error), start the ads SDK, then load ads.
4. Turn analytics on or off from the consent status.
5. In Settings, show "Privacy options" only if required; it reopens the form.

## Rules

- **No ad request before consent is resolved.**
- Child-directed games: tag requests for child treatment, no personalized ads, no advertising-ID use,
  different store declarations — confirm with the user.
- Store forms (data safety, advertising ID, app content) must match the SDKs actually in the build;
  check them against Google Play's and Apple's own publishing docs (store publishing is out of scope here).
- Never send device IDs, emails or player names to third parties outside the declared SDKs.
- Any consent/ads plugin is third-party code: Claude reviews it with the user before installing.

## Implementation — Godot 4

- Inside the ads autoload: `_gather_consent()` → `_start_mobile_ads()`; `privacy_options_required()`,
  `show_privacy_options()`.
- Use the UMP classes of the AdMob plugin, looked up by class name so the project still runs without
  the plugin.
- The analytics wrapper is enabled/disabled from the consent status.

## Other engines

Unity: Google Mobile Ads UMP API — `ConsentInformation.Update`,
`ConsentForm.LoadAndShowConsentFormIfRequired`, `ConsentInformation.PrivacyOptionsRequirementStatus`.

## Done when

- [ ] With an EEA test device, the form shows before any ad request.
- [ ] Settings shows "Privacy options" only when required, and it reopens the form.
- [ ] Analytics stays off when consent denies it.
- [ ] Legal questions are answered by the user or marked `TODO(confirm)`.
