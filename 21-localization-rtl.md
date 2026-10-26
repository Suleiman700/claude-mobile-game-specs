# 21 — Localization and right-to-left

> Play the game in the player's own language, including Arabic and Hebrew right-to-left layouts.

## What it is

All player-visible text comes from a translation table instead of being written in code, the game
picks the device language on first launch, and a Language option in Settings lets players switch.
Right-to-left (RTL) languages mirror the layout.

## Why it matters

Most mobile players don't speak English as a first language. Translated games get more installs,
better store ranking in each country, better reviews and longer play. Arabic and Hebrew markets need
proper RTL; a mirrored, readable UI is a big trust signal there.

## When to use it

- Any game published worldwide.
- Start with the strings in a table from day one, even if only English ships first; adding languages
  later is then cheap.
- Skip only for prototypes or games with almost no text.

## When it happens

- First launch: device language detected automatically (no setup screen).
- Anytime: Settings → Language (`10`).
- Push and local notifications use the same language (`13`, `14`).

## Before you build (for Claude)

1. Confirm which languages the game ships with; ask if unsure.
2. Show **Defaults** and ask: *"Use these values, or change any?"* Wait for the answer.
3. **Ask the user:**
   - the language list;
   - who reviews translations (machine translation needs a native check);
   - whether the logo/title is translated or kept in English.

## Defaults

| Setting | Default | Why |
|---|---|---|
| Languages | English, Arabic, Hebrew, Spanish, Portuguese, French, German, Russian, Chinese (Simplified) | Large mobile markets |
| Default language | device language, fallback English | No setup step |
| Settings | Language row (`10`) | Player control |
| Strings | one table (CSV or `.po`) keyed by IDs; no visible text hard-coded in scripts | Easy to translate |
| RTL | Arabic/Hebrew mirror the layout: left/right swap, arrows flip, numbers stay left-to-right | Natural reading |
| Fonts | the main font + **fallback fonts** for Arabic, Hebrew, CJK (e.g. Noto Sans Arabic / Hebrew / SC) | No missing-glyph boxes |
| Numbers | localized thousands separators; the game's currency symbol kept | Readable amounts |

## How it works

```
Launch → read device locale → set language (or saved choice) → load translations
UI text = tr("KEY") → font picks glyphs from the main font or a fallback
RTL language → root controls switch to right-to-left layout direction
```

Example of mirroring:

```
LTR:  (★3)[████ 45%]                [II]
RTL:  [II]                [45% ████](3★)
```

## Rules

- Leave room for longer text: German/Russian are ~30% longer; buttons wrap or shrink text, never clip.
- Never build sentences from pieces ("Hire" + "a cleaner"); use full templates with placeholders
  (`"Hire {name}"`).
- Plurals and gender handled by the translation system, not by code.
- Store listing translated separately (store publishing is out of scope here).
- Screenshot-test English, Arabic and German at the tall phone size (`17`).

## Implementation — Godot 4

- CSV translations imported as `.translation` files, registered in Project Settings; `tr("KEY")` in code.
- `TranslationServer.set_locale()` from the saved choice or `OS.get_locale_language()`.
- `Control.layout_direction = LAYOUT_DIRECTION_RTL` on the UI root for RTL languages.
- The theme's `FontVariation` lists fallback fonts so every script renders.

## Other engines

Unity: Localization package (String Tables, Smart Strings), TextMeshPro fallback font assets, an RTL
text plugin for Arabic/Hebrew shaping.

## Done when

- [ ] Switching language in Settings updates every screen live.
- [ ] Arabic and Hebrew mirror correctly and render without missing glyphs.
- [ ] Nothing clips in German at the tall phone size.
- [ ] No visible text is hard-coded in scripts.
