# PACA demo site — Proteus // Breakthrough

Interactive companion site for **Martin (2026), "Dodging Proteus"**: an arcade
recreation of the Protean Adversarial Choice Assay where readers can watch the
study's LLM subjects play against Proteus (replayed verbatim from the released
data) and play against Proteus themselves.

## Structure

- `index.html` — landing page
- `watch.html` — replay engine: all 14 model × frame cells, one representative
  released session each (chosen as win rate closest to the cell mean), with
  Proteus's per-trial channel-lock states recomputed exactly from history via
  the committed algorithm and validated against the released `sessions.csv`
  (win rate, P(T), runs z, activation rate: 98/98 sessions match)
- `play.html` — the playable game: 200 trials, TARGET LOCK indicator, animated
  Brown–Rosenthal debrief (Figs 3A/3B/3C/4), and a "you vs the models" field
  placement screen (press F after the debrief)
- `data/data.js` — bundled session + summary data (works over `file://`)
- `data/summary.json`, `data/sessions/*.json` — same data as plain JSON

## Run locally

Any static server works; no build step:

    python3 -m http.server 8000
    # open http://localhost:8000

Opening `index.html` directly from disk also works (data is bundled as JS,
not fetched).

## Regenerating data

`curate.py` (in the parent working directory) reads `trials_compact.csv` from
the [dodging-proteus](https://github.com/chimpanzity/dodging-proteus) repo,
recomputes per-trial Proteus channel state and all diagnostics, validates
against `sessions.csv`, and writes `data/`.

## Role mapping (important)

In the lab task the model was the **matcher** (won by matching Proteus's
choice); in the arcade the subject wins by **mismatching** (breakthrough).
Replays therefore draw the subject's real choices as lanes, draw subject wins
as breakthroughs, and mirror Proteus's displayed lane — an isomorphism that
preserves every statistic (win rate, marginals, runs, WSLS, channel
activations). Noted in the site's fine print.
