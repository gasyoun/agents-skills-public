---
name: "cologne-pages-landing"
description: Deploy the SEO/UX-optimised GitHub Pages landing page (index.html + .nojekyll) to Cologne dictionary repos missing it; discovery/template scripted (cologne_pages_landing.py), per-dict facts stay LLM/human. Sonnet 5 (claude-sonnet-5) tier. Do not use for full repo bootstrap — that is /cologne-onboard-repo; this only deploys the GitHub Pages landing page.

---

# cologne-pages-landing

Use this skill when the user asks to run the migrated source command `cologne-pages-landing`.

## Command Template


# /cologne-pages-landing <REPO|all>

_Created: 06-07-2026 · Last updated: 05-09-2026_

## Default path (seconds — run this first)

```sh
python ~/Documents/GitHub/Uprava/tools/cologne_pages_landing.py --dry-run
```

LLM / judgment residue only after the script report exists (or the script exits non-zero with a real defect).

Give every Cologne dictionary repo a proper GitHub Pages front door: a single
self-contained `index.html` (SEO meta + Open Graph + JSON-LD `Book` schema +
styled hero/CTA layout) plus `.nojekyll` so Pages serves it statically.

**Default path — run the script, don't hand-roll discovery again.**
[`Uprava/tools/cologne_pages_landing.py`](https://github.com/gasyoun/Uprava/blob/main/tools/cologne_pages_landing.py)
(H2790, Sonnet 5 — ZJ pages-landing deploy) is the mechanical port of Phases 0/3 below: it set-diffs local
`sanskrit-lexicon/*` clones for a missing SEO landing, applies the same
csl-*/tooling skip-rules, and — for any repo whose facts are already filled
into its `DICT_FACTS` table — opens the PR via the same chassis as
[`cologne_batch_deploy.py`](https://github.com/gasyoun/Uprava/blob/main/tools/cologne_batch_deploy.py).

```sh
cd ~/Documents/GitHub/Uprava
python tools/cologne_pages_landing.py --dry-run          # set-diff, read-only (default)
python tools/cologne_pages_landing.py --selftest         # fixture + GRA/MD template round-trip
python tools/cologne_pages_landing.py --pr --repo <CODE> # open the PR (needs DICT_FACTS[<CODE>] filled first)
```

A repo the script reports `NEEDS_FACTS` has no `index.html` but no
`DICT_FACTS` entry yet — do Phase 2 below by hand (or LLM), add the entry to
`DICT_FACTS` in the script, then re-run `--pr --repo <CODE>`. A repo reported
`NEEDS_REVIEW` has a real non-SEO `index.html` — read it before touching
anything; the script never overwrites it.

Gold-standard reference implementations (structure only — do NOT redesign):

- `~/Documents/GitHub\GRA\index.html` (German-title dict)
- `~/Documents/GitHub\MD\index.html` (English-title dict)

The phases below are the manual/fallback procedure the script encodes —
read them to understand *why* the script does what it does, or when the
script itself needs a fix, not as the everyday runbook.

---

## Phase 0 — Target discovery (script does this — `--dry-run`)

Argument = one repo code (e.g. `SKD`) or `all`.

Exclusions (mirrored in the script's `TOOLING_SKIP` set + csl-* prefix rule):
- **Skip tooling/infra repos** (csl-pywork, csl-orig, csl-corrections,
  csl-apidev, csl-websanlexicon, cologne-data, csl-* generally) — this skill is
  for *dictionary* repos whose Pages site is a landing page, not an app.
- **Skip repos with a non-SEO `index.html` that is a real app/site** (check
  before overwriting — if `index.html` exists but has no `og:site_name`, READ
  it first; only replace trivial redirect/placeholder pages, never a
  functioning site).
- csl-orig is never touched (org rule).

## Phase 1 — Publish-safety gate (cheap form)

These repos are already public and the landing page contains only
already-public bibliographic facts, so the full `/publish-safety-check` is
normally not needed. Run it ONLY if the target repo is private or the page
would surface anything beyond title/author/year/description/scan-link.

## Phase 2 — Gather per-dictionary facts

For each target, collect (from the repo README, `prefaces/`, and
[CDSL](https://www.sanskrit-lexicon.uni-koeln.de/)):

| Fact | Example (GRA) |
|---|---|
| Full title | Wörterbuch zum Rig-Veda |
| Code badge | GRA |
| Author + place, year | Hermann Grassmann · Leipzig, 1873 |
| One-sentence description | "…complete concordance-dictionary of the Ṛgvedic vocabulary…" |
| Canonical URL | `https://sanskrit-lexicon.github.io/GRA/` |
| Browse URL | `https://www.sanskrit-lexicon.uni-koeln.de/scans/GRAScan/2020/web/webtc/indexcaller.php` |
| License | CC-BY-SA-4.0 (per RH1 policy; check the repo's actual LICENSE) |

Verify the Browse URL actually resolves (HTTP 200) before baking it in — the
`<CODE>Scan/2020/web/webtc/indexcaller.php` pattern holds for most but not all
dictionaries. If Cologne is unreachable, check
[`Uprava/SERVER_OUTAGES.md`](https://github.com/gasyoun/Uprava/blob/main/SERVER_OUTAGES.md)
before concluding the URL is wrong.

## Phase 3 — Deploy (script does this — `--pr`)

Per target repo:

1. Write `index.html` from the gold-standard template with the facts
   substituted (title/desc in `<title>`, `meta description`, OG, Twitter,
   JSON-LD, hero). Keep `og:image` = `https://sanskrit-lexicon.github.io/cdsl-card.png`.
   The script's `TEMPLATE`/`build_landing_html()` does this from `DICT_FACTS`.
2. Write empty `.nojekyll` at root (this was a follow-up fix commit in every
   first-wave repo — bake it in from the start).
3. Opens a PR on branch `cologne-pages-landing/<CODE>` via the
   `cologne_batch_deploy.open_file_pr` chassis (GitHub-API writes, not a local
   commit+push — see that module's docstring for why).
4. Check Pages is enabled: `gh api repos/sanskrit-lexicon/<REPO>/pages`.
   If 404, enable from the default branch root:
   ```sh
   gh api repos/sanskrit-lexicon/<REPO>/pages -X POST \
     -f "source[branch]=<default>" -f "source[path]=/"
   ```
5. After Pages builds, verify the live URL returns the new page (poll a
   couple of minutes max; if the build lags, log it and move on).

Cap parallelism at 4. Continue past per-repo errors; log DONE/ERROR per repo.

## Phase 4 — Roll-up + hygiene

- Emit a table: `| Repo | Result | Live URL | Commit |`.
- Append to each touched repo's `.ai_state.md` `## ✅ Completed`:
  `- DD-MM-YYYY: /cologne-pages-landing — SEO landing + .nojekyll live`.
- If this was an `all` run, note remaining skips (real-app index.html repos,
  unreachable scan URLs) so the next run knows they were deliberate.

---

## Notes for future maintenance

- The page is one self-contained file: inline CSS, no JS, no external assets
  except the shared OG card image. Keep it that way.
- If the shared design changes, update GRA/MD first (they are the reference),
  then re-run this skill in explicit-replace mode (a human decides when).
- `/cologne-onboard-repo` should include this skill for any newly adopted
  dictionary repo.

_Dr. Mārcis Gasūns_
