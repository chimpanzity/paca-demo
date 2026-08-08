# HANDOVER — PACA demo site ("Proteus // Breakthrough")

Pick-up notes for continuing this project in Claude Code on the laptop.
Last updated at the end of a desktop session.

---

## What this is

Interactive companion site for **Martin (2026), "Dodging Proteus"** — an arcade
recreation of the Protean Adversarial Choice Assay (PACA). Readers can **watch**
the study's LLM subjects replay real sessions against Proteus, or **play** against
Proteus themselves. Pure **static site — no build step, no dependencies, no server
required** (study data is bundled as JS so it works over `file://`).

**Headline finding (used as the site's hook):** informed-frame models averaged a
**34.7%** win rate vs **45.6%** for minimalist-frame — **6 of 7 models played
*worse* when told the optimal (equilibrium) strategy.**

---

## Where it lives / git status

- Folder (desktop): `C:\PACA_Website\paca-site\paca-site\`
- Git: initialized, branch `main`, committed. Remote `origin` →
  `https://github.com/chimpanzity/paca-demo` (**private**).
- **NOT pushed yet.** The empty repo must be created on GitHub first
  (github.com/new → owner `chimpanzity`, name `paca-demo`, Private, no README/
  gitignore/license), then: `git push -u origin main`.
- On the laptop: either `git clone` the repo (once pushed) or unzip
  `proteus-breakthrough-site.zip` (was placed in the desktop's Downloads).

---

## How to run

- **Quick:** double-click `index.html` — works over `file://` (data is bundled).
- **Full fidelity:** from the folder, `python -m http.server 8123`, open
  `http://localhost:8123/`.
- **Cache gotcha:** browsers cache hard. After an edit, hard-refresh (Ctrl+F5) or
  append a throwaway query string, e.g. `play.html?v=2`.

---

## File map

| File | Purpose |
|---|---|
| `index.html` | Landing page: logo masthead, scoreboard stat (34.7 vs 45.6), two CTA cards (Watch / Play), "how the arcade maps" list, fine print, paper links. Has `og:` tags + screen-reader `<h1>`. |
| `play.html` | The playable game — you vs Proteus, canvas arcade with the full live telemetry rack. |
| `watch.html` | Replay engine — model×frame selector, plays released sessions verbatim, same telemetry rack, updating each trial. |
| `data/data.js` | `window.PACA_DATA` = `{summary.cells, sessions}`. Bundled so `file://` works. |
| `data/summary.json`, `data/sessions/*.json` | Same data as plain JSON. |
| `logo.png` | 1448×1086 neon logo (Proteus → bull → fish). |

---

## Layout of the two game pages (play & watch are near-identical)

Three columns inside `#wrap` (flex, centered): **LEFT `#leftrack` │ canvas cabinet
(520×780) │ RIGHT `#side`**. Both side columns hide below 940px; `fit()` reserves
656px for the pair and scales the canvas to fit between them.

**LEFT column (top→bottom):**
- Logo (screen-blended so its black bg vanishes)
- **SHOT TAPE** — barcode of last 32 shots; `▲:` / `★:` row headers (top lane =
  Triangle/left, bottom = Star/right); gold square = breach, red = intercept.
- **CHOICE + OUTCOME** — one panel, two aligned rows: choices (`▲★`, contiguous)
  on top, outcomes (`1` gold = breach / `0` red = intercept) directly below each
  choice. Rendered as 6px cells + 2px gap behind a 16px gutter so columns line up
  with the shot-tape squares. Last 32 trials.
- **CHANNEL THREAT BOARD** — Proteus's 9 detection channels with live p-values,
  redline at p=.05, plus a "last commit" line (min-height locked so it can't jitter).

**RIGHT rack (top→bottom):** FIG 3A win-rate spark → FIG 3B marginal balance →
FIG 3C runs-test z → **FIG 4 stay/shift** (enlarged ~full width, solid quadrant
dividers) → SESSION stats.

**Canvas:** end zone up top (gold, with a `BREACHED` counter = breakthroughs),
Proteus turrets (`★` left / `▲` right), player silos (`▲` left / `★` right), two
lanes. Title screen (play) / idle "press ▶" card (watch) show the logo.

---

## Core mechanics & conventions

- Proteus = **Lee, Conroy, McGreevy & Barraclough (2004), Algorithm 2**, intercept-
  modified. Choice chars: **`T` = LEFT = Triangle**, **`S` = RIGHT = Star**.
  Reward chars in `pastCR` (subject POV): **`w` = broke through (win)**, `l` = intercepted.
- Arcade rule: **same lane = interception** (Proteus wins); **different lanes =
  breakthrough** (you win). This is the role-flip from the lab task (model was the
  *matcher*); a pure relabeling that preserves every statistic.
- **Shape overlay logic:** player silos `▲`(L)/`★`(R); Proteus turrets are the
  **opposite** `★`(L)/`▲`(R). Net effect: **shape MATCH = breakthrough, shape
  MISMATCH = interception** — restores the paper's "match = win" framing on top of
  the lane mechanic.
- **watch.html replay:** choices/wins come from released data verbatim; the channel
  board is recomputed **live** from accumulated history via `computeAllChannels`.
  Verified: recomputed lock channel matches the recorded lock on **0/2800**
  mismatches across all sessions — so board and TARGET-LOCK reticle always agree.
- Telemetry render fns: `renderChannels`, `renderShots`, `renderTapes`, `renderRate`
  (which draws `drawSpark`/`drawFig3B`/`drawFig3C`/`drawFig4`), wrapped by
  `renderPanels`. Called in `resolve()` each trial and on reset/load.

---

## ⚠️ Editing gotchas (learned the hard way on Windows)

1. **`play.html` and `watch.html` are near-duplicates** — almost every change must
   be applied to **both**. Their Proteus core, telemetry rack, canvas drawing, and
   game-over/debrief code are shared by copy-paste.
2. **Windows Python ignores MSYS paths.** The interpreter here is Windows Python; it
   does **not** understand `/c/...` paths and resolves relative paths against
   `C:\Users\...\` (or the primary working dir), not the site folder. Edits were done
   with Python scripts using **absolute `C:/PACA_Website/...` paths**. Do the same, or
   use the Edit tool.
3. **Bash cwd drifts** between calls — never trust relative paths; pass absolute.
4. **Browser preview is limited:** the in-app pane blocks `localhost` and only renders
   `file://` as a non-interactive static snapshot (no screenshot/console). To verify,
   read the files / `curl` the server, or open in a real browser.
5. **Unicode in Python one-liners:** writing files is fine with
   `open(..., encoding="utf-8")`, but `print()`-ing `▲ ★ → ↑` to the console throws
   (cp1252 stdout). Don't print unicode; print counts/booleans instead.
6. Edits in this session were made as **assertion-checked string replacements** (each
   anchor asserted to occur exactly once) to stay safe against the duplication.

---

## Done this session (high level)

- Rebuilt `index.html` (scoreboard hero, arcade CTA cards, CRT overlay, mobile
  breakpoint, favicon, logo masthead, og tags).
- Swapped in the live-telemetry `play.html`; re-merged the "F · you vs the models"
  compare screen; locked the commit-line height.
- Ported the telemetry rack into `watch.html` (updates during replay).
- Added end-zone `BREACHED` counter; logo title/idle cards on both game pages.
- Added the LEFT column; player-silo shapes; opposite Proteus-turret shapes.
- Enlarged FIG 4 with prominent quadrant dividers.
- Reorganized the left column: shot tape (with `▲:`/`★:` headers) + combined,
  column-aligned choice/outcome two-row panel + channel board.
- `git init` + first commit + `origin` remote; created transfer zip.

---

## Suggested next steps / open threads

- **Finish the push:** create the empty private `paca-demo` repo, then
  `git push -u origin main`.
- Private repo + **GitHub Pages** needs a paid plan; flip to Public for the free live URL.
- The **left column has empty space** below the channel board — reserved for future content.
- Optional: draw the launched missile as the player's lane shape and the countermeasure
  as Proteus's shape, so match/mismatch is legible **mid-flight**, not just at the batteries.
- Optional: label the choice/outcome panel's blank left gutter (currently just an
  alignment spacer).
- Consider deleting/gitignoring this `HANDOVER.md` before any public release — it's a
  dev note, not site content.
