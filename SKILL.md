---
name: mobile-game-specs
description: Build mobile game systems from tested specs (design language, buttons, popups, HUD, money feedback, juice, touch controls, splash, menu, settings, ads, rate prompt, notifications, push, retention, dev panel, testing, consent, IAP, analytics, localization/RTL, saves, tutorial, Android lifecycle, feature switches, accessibility, performance, upgrades). Use when starting a mobile game or adding/building one of these features.
---

# mobile-game-specs

A library of 28 build specs for polished mobile games (Godot 4 first, notes for Unity and web). Each spec says what the system is, which games need it, the default values, the rules, an implementation outline, and a "Done when" checklist. `README.md` has the full file list, the genre table and the build order.

## Pick the files
Read only what the request needs.
- **One feature** ("add daily rewards"): its file (`15-retention.md`), plus any file it references for parts it touches (e.g. `03` for the popup, `05` for the coins).
- **New game**: look up the genre in README → "Which files for which game", then follow README → "Order for a new game", one group at a time.
- **Every game shipping on Android** also needs 08, 17, 18, 22, 24, 26, 27. Mention any that are missing once; don't build them unasked.

## Where specs and decisions live
1. **Project overrides first.** If the game repo has `docs/game-prompts/NN-*.md`, those copies win over this skill's copies.
2. **Decisions file.** `docs/game-prompts/project-decisions.md` in the game repo records every confirmed answer, per spec:
   ```
   ## 11-ads (confirmed 2026-10-26)
   - Interstitial every 8 purchases (default 5) · banner on menu: no (default yes) · rewarded: double gift, free cash
   - Remove-ads purchase: yes · child-directed: no
   ```
   Read it before asking anything. Only ask what it doesn't answer. Create it if missing; never overwrite another spec's section.

## For every spec (the four steps from README)
1. **Confirm the game needs it**; ask if the genre makes it unclear.
2. **Show the Defaults table** (with any values already in project-decisions.md) and ask "Use these values, or change any?", then wait.
3. **Ask only the spec's listed questions.** Record the answers in project-decisions.md.
4. **Build it**, then prove every "Done when" item: tests per `17`, captures at 9:16 and 9:19.5.

## Hard rules
- Never install third-party plugins or SDKs (ads, billing, notifications, Firebase, review) without reviewing source, license and downloads with the user first.
- Legal answers (children audience, privacy policy, consent regions) come from the user. Unanswered ones become `TODO(confirm)`, never a guess.
- Real ad IDs, product IDs and keys live in git-ignored config, never in code.
- Tests and captures never touch the player's real save, settings or ad IDs.
- Report honestly: failing checks are shown, partial work is called partial.

## Review handoff
After step 4's captures, if the `harsh-design-critic` skill is installed, offer one line: "Run a harsh design review on these screens?" On yes, the critic grades the screens against these specs and project-decisions.md. Its findings come back as approvable questions; nothing changes until the user approves.
