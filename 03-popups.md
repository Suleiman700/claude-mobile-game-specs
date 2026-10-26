# 03 — Popups

> One popup look for the whole game (ribbon title, round close, pop-in), plus bottom sheets, toasts, tip pills and the "you got something new" reveal.

## What it is

Every message, reward, menu and confirmation that appears over the game uses the same card: a dim
backdrop, a panel with a colored title **ribbon** across its top edge, a round ✕, content, and big
buttons. Related surfaces: the **bottom sheet** (long lists like upgrades or shop), the **toast**
(short info that fades), the **tip pill** (hints), and the **reveal popup** that shows what the
player just unlocked or bought.

## Why it matters

- Players learn one pattern and then read every popup instantly.
- A clear main button and a clear "no thanks" button avoid mis-taps.
- Showing what was unlocked makes every purchase feel like a reward, not just a number going down.

## When to use it

- **Always**, in any game with rewards, menus or confirmations.
- The reveal popup: games where the player unlocks or buys **new things** (workers, rooms, areas,
  characters, weapons, skins, levels). Skip it for pure score games with nothing to unlock.

## When it happens

- Startup: welcome back → daily reward → others, one at a time.
- During play: rewards, reveals, pause, settings, shop/upgrades.
- End of a level or run: results, then (sometimes) the rate prompt (`12`).

## Before you build (for Claude)

1. Confirm which popups the game needs (list below) and ask if unsure.
2. Show **Defaults** and ask: "Use these values, or change any?" and wait.
3. **Ask the user:**
   - the game's popup list (table in "How it works", trimmed or extended);
   - the reveal popup ribbons (NEW HIRE, NEW ITEM, UNLOCKED...) and their benefit texts.

## Defaults

| Setting | Default | Why |
|---|---|---|
| Card width | `620` | Fits every phone with margins |
| Backdrop | dark dim, α `0.6` | Focus |
| Open animation | card scale `0.72 → 1` in `0.26 s` (back-ease), dim fade `0.18 s` | Lively, not slow |
| Open sound | soft `pop` (not a whoosh) | Pleasant when repeated |
| Tap outside closes | `no` (only ✕ or a button) | No accidental closes |
| One popup at a time | `yes`; new ones queue | Never stacked |
| Game time while open | timers that punish the player freeze (patience, combo); the world keeps animating | Shopping never punishes |
| Toast | dark pill near the top, `2.6 s` then fades | Short info only |

## How it works

### Anatomy

```
        ┌──────────────┐              (●✕)   ← round close, 76 px, top-right
 ┌──────│   RIBBON     │───────────────┐     ← ribbon title overlaps the card top by 46 px
 │      └──────────────┘               │
 │   picture (optional, 130–150 px)    │
 │   text / content                    │     ← body: vertical list, 18 px gaps
 │   [ main action (colored, 100) ]    │
 │   [ neutral action ]                │
 └─────────────────────────────────────┘
```

Panel: `PANEL` color, 7 px white rim, radius 40, soft shadow. Rows inside are `CARD`s; a row ready
to claim is a white card with a green border.

### Typical popups

| Popup | ✕ | Buttons |
|---|---|---|
| Pause | yes (= resume) | Resume · Settings · Main menu · (Developer in debug) |
| Settings | yes | — (see `10`) |
| Gift / reward | no | Double it ▶ ad · Nice! |
| Welcome back (offline) | no | Collect 2× ▶ ad · Collect |
| Daily reward (on start) | no | Claim |
| Daily reward (from button) | yes | Claim (or "claimed today") |
| Goals / missions | yes | per row: GO / CLAIM / DONE |
| Reveal (new thing) | yes | Nice! |
| Free <currency> | yes | Watch ▶ (or cooldown text) |
| Level / run complete | no | Continue (rate prompt may follow) |
| Confirm destructive | yes | Danger confirm · Cancel |

### Reveal popup (show what you got)

Whenever the player **unlocks or buys something new**, show that exact thing:

```
          ┌──────────────┐        (✕)
   ┌──────│  NEW ITEM    │─────────┐      ← NEW HIRE / NEW ROOM / UNLOCKED / NEW WEAPON...
   │        ( picture )            │      ← the actual thing in a round disc, 150 px:
   │                               │         live 3D portrait / model render / icon
   │      <Item> unlocked!         │      ← outlined title in the accent color, 40 px
   │  <What it does for you.>      │      ← one plain line about the benefit
   │  [ ✓        Nice!          ]  │      ← Green, check icon
   └───────────────────────────────┘
```

| Setting | Default |
|---|---|
| Shown for | `first unlock of each kind` (new worker type, room, area, character, weapon, shop item); not for repeats (the 3rd copy of the same worker) |
| Picture | live 3D portrait for characters (idle animation), render or icon for objects |
| Sound + haptic | `happy` jingle + medium haptic; confetti for big unlocks (new area/level) |
| Upgrades (levels of something owned) | no popup: the row animates instead (`28`) |

Text says what it is and the benefit in one plain line, never stats only.

### Other surfaces

- **Bottom sheet** (Upgrades, Shop): slides up to ~62% of the screen, ribbon over its top edge, ✕,
  tabs, scrolling rows. The top of the game stays visible.
- **Toast:** short info ("You earned a star!"). Never for anything the player must act on.
- **Tip pill:** white pill with an info icon above the main bottom button (tutorial, hints).

### Ask budget (notifications, rating, offers)

Three systems ask the player for something from session 3: the notification soft-ask (`13`), the rate prompt
(`12`) and the starter pack offer (`19`). They never pile up:

| Setting | Default |
|---|---|
| Max asks per session | `1` of these three combined |
| Priority when several are due | notification soft-ask → rate prompt (needs a good moment) → starter pack |
| Never | during the tutorial, after a failure, right after an ad, while startup popups are queued |
| Storage | `last_ask_session` in the settings/meta save |

An ask that loses the slot waits for the next session; its own retry timer doesn't restart.

## Rules

- A popup that pays <currency> sends the coins from **its button** to the counter (`05`).
- "Double it" buttons show only when a rewarded ad is ready; if the ad fails, pay the base amount.
- Startup popups queue: welcome back → daily reward → reveals; never on top of each other.
- Every popup is reachable and closable on a 9:19.5 phone with the banner ad visible.
- Popups work while the game is paused.

## Implementation — Godot 4

`UiKit.modal(parent, title, on_close, width, closable) -> {root, body}` builds the card; a HUD-level
`PopupManager` keeps one open at a time, queues the rest, and exposes `is_open()` for freezing
timers. Popups use `process_mode = PROCESS_MODE_ALWAYS`. Reveal: `show_reveal(kind, picture, title, benefit)`.

## Other engines

- Unity: one Modal prefab with header/body/footer slots and a PopupManager with a queue.

## Done when

- [ ] All popups share width, ribbon, ✕ position and animation (side-by-side screenshots).
- [ ] Startup popups never overlap; closing one reveals the next.
- [ ] Unlocking a new kind of thing shows its reveal popup with picture, name and benefit.
- [ ] At most one of soft-ask / rate prompt / starter pack appears per session.
