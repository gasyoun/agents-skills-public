---
name: "review-sheet-restandardize"
description: 'Port a repo''s EXISTING review-sheet builders onto the current shared emitter standard (csl-pyutil render_review_sheet), rebuild still-unvoted sheets, reconcile REVIEW_SHEETS_INDEX.md. Triggers: "restandardize the sheets". Sonnet 5 (claude-sonnet-5) tier.'

---

# review-sheet-restandardize

Use this skill when the user asks to run the migrated source command `review-sheet-restandardize`.

## Command Template


# /review-sheet-restandardize [repo | all]

_Created: 20-07-2026 · Last updated: 21-08-2026_

## Default path (seconds — run this first)

```sh
python ~/Documents/GitHub/Uprava/tools/review_sheet_restandardize.py --repo PATH
python ~/Documents/GitHub/Uprava/tools/review_sheet_restandardize.py --all
python ~/Documents/GitHub/Uprava/tools/review_sheet_restandardize.py --repo PATH --rebuild-unvoted
python ~/Documents/GitHub/Uprava/tools/review_sheet_restandardize.py --repo PATH --reconcile-index
python ~/Documents/GitHub/Uprava/tools/review_sheet_restandardize.py --repo PATH --assert-markers
python ~/Documents/GitHub/Uprava/tools/review_sheet_restandardize.py --selftest
```

LLM / judgment residue only after the script report exists (or the script exits non-zero with a real defect).

[`Uprava/tools/review_sheet_restandardize.py`](https://github.com/gasyoun/Uprava/blob/main/tools/review_sheet_restandardize.py) ([H3223](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H3223-Grok_Uprava_zj-review-sheet-restandardize_21.08.26.md) — Grok 4.5, zj-review-sheet-restandardize) inventories builders (`build_*review*.py` / `review_sheet_standard.py`) and `review/` sheets, classifies each sheet voted / partial / unvoted from sibling `*_decisions.json`, and dry-runs a table of builder path · sheet path · vote class · index row present/stale/missing. `--rebuild-unvoted` re-emits **unvoted** HTML through `csl_pyutil.render_review_sheet` and **refuses** a sheet that already has `*_decisions.json` unless `--ids-stable` is explicit. `--reconcile-index` rewrites stale worktree `file:///` paths in [REVIEW_SHEETS_INDEX.md](https://github.com/gasyoun/Uprava/blob/main/REVIEW_SHEETS_INDEX.md) when the filename still exists on disk. `--assert-markers` greps `color-scheme` / `STORE_KEY` / `idchip`. Phase 2 (port a hand-rolled builder onto the shared emitter) stays LLM — do not fold it into the script.

The phases below are the manual/fallback procedure the script encodes — read them when the script itself needs a fix, not as the everyday runbook. Phase 2 is still the judgment gate.

When the review-sheet standard bumps, every repo that grew its own sheet builder lags it at
once — and the migration got re-derived separately per repo, in different languages, in the
same week: a JS multi-option template in csl-atlas
([H1314](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1314-Opus_csl-atlas_review-sheets-standard-port_19.07.26.md) — Opus 4.8, csl-atlas review-sheets standard port),
a python `review_sheet_standard.py` in SanskritLexicography
([H1313](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1313-Fable_SanskritLexicography_review-standard-v030-orgwide-remake_19.07.26.md) — Fable 5, review-standard v0.3.0 org-wide remake),
again in SanskritGrammar
([H1315](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1315-Opus_SanskritGrammar_review-sheets-standard-port_19.07.26.md) — Opus 4.8, SanskritGrammar review-sheets standard port)
and RussianTranslation
([H1301](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1301-Opus_RussianTranslation_pwg-ru-review-sheet-ux-standard-regen_19.07.26.md) — Opus 4.8, pwg-ru review-sheet UX standard regen).
[`/review-sheet`](https://github.com/gasyoun/claude-config/blob/main/commands/review-sheet.md)
owns *generating a fresh sheet on the standard*; it owns neither migrating an existing
builder nor reconciling the index. This skill is that other half. It fires on a **standard
bump**, not routinely.

Argument: the repo to port, or `all` to sweep every repo with a sheet builder.

## Phase 1 — Inventory the builders and the live sheets

In the target repo(s), find every sheet-producing path and every sheet already on disk:
builder scripts (`build_*_review*.py`, `review_sheet_standard.py`, `.mjs`/JS templates with
a `sheetHtml`-style function), the `review/` tree, and the repo's rows in
[`Uprava/REVIEW_SHEETS_INDEX.md`](https://github.com/gasyoun/Uprava/blob/main/REVIEW_SHEETS_INDEX.md).
Classify each sheet **voted / partially-voted / unvoted** by looking for its
`*_decisions.json` — this classification governs Phase 3 and is the one step that must not
be skipped.

## Phase 2 — Port the builder to the current standard

Read `/review-sheet` for the standard as it stands now, then:

- **Python repos** — delete the hand-rolled markup and call
  `render_review_sheet(items, config)` from
  [`csl-pyutil`](https://github.com/sanskrit-lexicon/csl-pyutil), passing every mandatory
  config key. Map the repo's old item shape onto the current one; drop fields the emitter no
  longer takes rather than smuggling them through.
- **JS-stack repos** (csl-atlas) — the shared emitter is python, so mirror the mandatory
  features and the dark-theme contrast requirement in the local template, and leave a pointer
  saying the mirror is temporary until that repo's port to the emitter lands. A mirror that
  does not say it is a mirror becomes a second standard.

## Phase 3 — Rebuild only what is safe to rebuild

**Rebuild unvoted sheets. Never rebuild a sheet that already has votes** unless item IDs are
provably stable — regenerating under changed IDs orphans a human's completed pass, which is
strictly worse than an old-looking sheet. Partially-voted sheets: rebuild only with stable
IDs preserved, and say in the report that votes were carried; otherwise leave and flag for a
human. This is the one destructive risk in the whole skill.

## Phase 4 — Reconcile the sheet index

Fix the repo's rows in `REVIEW_SHEETS_INDEX.md`: stale paths (a per-handoff worktree that no
longer exists is the common case — H1314), sheets present on disk with no row, rows whose
sheet is gone, and statuses that no longer match (`Awaiting vote` on a sheet already applied).

## Phase 5 — Assert the markers, then commit

Verify mechanically rather than by eye — grep each rebuilt sheet for the mandatory standard
markers (the required config keys' rendered output and the dark-theme `color-scheme`
declaration). A port that "looks right" but is missing a marker is the exact failure this
phase exists to catch. Report: builders ported, sheets rebuilt vs deliberately skipped (with
why), index rows fixed, markers asserted. Commit per the repo's write rules; the standard
bump itself is a `CHANGELOG` entry.

## When this isn't the right tool

- Generating ONE new sheet → `/review-sheet`.
- Applying a completed vote → `/decisions-apply`.
- Registering a freshly-made sheet → that is `/review-sheet`'s own index step.

## Compatibility Rule

The canonical source of this skill is `~/.claude/commands/review-sheet-restandardize.md` (the claude-config repo); Codex and Grok twins are GENERATED from it by
`scripts/generate_playbook_twins.py review-sheet-restandardize` and must never be hand-edited.

_Dr. Mārcis Gasūns_
