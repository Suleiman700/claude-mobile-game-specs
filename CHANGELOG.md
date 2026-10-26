# Changelog

## 1.0.0 — 2026-10-26
First published version.

- 28 specs covering UI, feel, menus, monetization, retention, quality and safety.
- Packaged as a Claude skill (`SKILL.md`): picks files by request and genre, remembers confirmed values in
  `docs/game-prompts/project-decisions.md`, lets project copies override the library, hands screenshots to
  `harsh-design-critic` for review.
- Touch targets: every tappable area ≥ 88×88 px; smaller visible shapes (plus button, dense list rows, DEV button)
  get invisible padding (`01`, `02`, `16`, `26`).
- Ask budget: the notification soft-ask, rate prompt and starter pack share one ask per session (`03`, `12`, `13`, `19`).
- Disabled vs hidden: can't do it yet → disabled; feature unavailable right now → hidden (`02`, `04`).
- Crash reporting section and `crash_reports` switch (`20`, `25`).
- `05` renamed to "Money feedback"; store publishing marked out of scope with pointers to the stores' own docs.
