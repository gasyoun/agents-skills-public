---
name: "paper-submission-pack"
description: 'Assemble a paper''s submission package from Uprava/ARTICLES.md — byline, CITATION.cff, cover letter, venue checklist, ACL Responsible-NLP checklist, DOCX export when required. Triggers: prepare Axx for submission, submission pack. Do not use for content review or rewriting — this only assembles the submission package (byline, CITATION.cff, cover letter); for content review use /paper-referee.'

---

# paper-submission-pack

Use this skill when the user asks to run the migrated source command `paper-submission-pack`.

## Command Template


# /paper-submission-pack

_Created: 03-07-2026 · Last updated: 05-09-2026_

Every submission currently rebuilds the same package by hand. This runbook assembles it once, correctly, from the canonical sources.

Argument: the stable paper ID (`Axx` / `Mxx`) from `Uprava/ARTICLES.md`, or enough title to resolve one.

## Phase 1 — Resolve the paper

Look it up in `Uprava/ARTICLES.md`: repo, file path, current readiness, target venue, blocker. Read the manuscript file itself — confirm it actually is at the claimed readiness before packaging (a 2/5 skeleton gets a "not ready" report, not a pack).

## Phase 2 — Byline + metadata

- **Paper byline** (from the identity memory, and note it is NOT the file-authorship line): Mārcis Gasūns — no "Dr." — independent scholar, ORCID `0000-0003-4513-884X`, gasyoun@ya.ru. Apply the venue's required format.
- `CITATION.cff` in the paper's repo: create or update (title, authors with ORCID, date, repo URL). Validate with `cffconvert --validate` if available.

## Phase 3 — Cover letter + checklist

In the paper's repo next to the manuscript (dated header + byline footer per the doc contract):

- `<id>_cover_letter.md` — skeleton addressed to the venue: contribution in 2–3 sentences, fit to the venue, novelty statement, no-dual-submission line, data/code availability with full blob URLs.
- `<id>_submission_checklist.md` — venue requirements (fetch the venue's author guidelines): word/page limits, abstract length, keywords, anonymization needs, figure formats, reference style, required statements (ethics, funding, competing interests), submission portal URL.

Flag every checklist item the manuscript currently fails — that list is the remaining work.

## Phase 3.5 — Checklist gate (ACL Responsible NLP + reproducibility, applied everywhere)

Adopted 11-07-2026 ([H666 (Sonnet 5) — claude-config ACL paper-checklists gate](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H666-Sonnet_claude-config_acl-paper-checklists_11.07.26.md)): the pack is **incomplete** without this phase, regardless of
venue — a missing Limitations section is a blocker, not a note. Fetch the current
checklist from [aclrollingreview.org/responsibleNLPresearch](https://aclrollingreview.org/responsibleNLPresearch/)
(don't reproduce from memory — cite the version/date fetched; last confirmed
11-07-2026, "Updated for ARR October 2024 cycle") and produce
`<id>_checklist.md` next to the manuscript: one line per item, `yes` / `no` / `n/a` +
a pointer (section/line) to where it's satisfied. The item families:

- **A. General** — A1 Limitations discussion (scope of claims, assumptions, robustness
  across datasets/languages/runs), A2 Risk discussion (harms, fairness, privacy,
  dual-use).
- **B. Scientific artifacts** — B1 citation of creators (version/URL), B2 licenses/terms
  per asset, B3 intended-use compatibility, B4 personal-info/offensive-content handling,
  B5 artifact documentation (language, domain, demographics), B6 dataset statistics
  (counts, splits).
- **C. Computational experiments** — C1 model/infra details (params, compute budget),
  C2 experimental setup (hyperparameter search, selection criteria), C3 descriptive
  statistics (error bars, single-run vs. mean), C4 package/tool version numbers.
- **D. Human annotators/participants** — D1 instructions text, D2 recruitment/payment,
  D3 consent, D4 ethics review status, D5 annotator demographics. Mark `n/a` outright
  for papers with no human-subjects component (most of this org's lexicographic work).
- **E. AI assistant usage** — E1 disclosure of AI-assisted drafting/analysis tools used
  in producing the paper.

A missing Limitations (A1) section or an unfilled B-family item for a paper that uses
external data/artifacts blocks the pack — report it as the top remaining-work item, not
a minor note.

**Venue calibration.** This org's default venues (lexicography, digital humanities —
see [ARTICLES.md](https://github.com/gasyoun/Uprava/blob/main/ARTICLES.md)) do not
formally require the ARR checklist artifact — but the underlying practices (state your
limitations, document your data provenance, report reproducibility details) are a good
internal quality bar regardless of venue, so apply the checklist review always. Attach
the *formal* filled-checklist file as a submission artifact only when the target venue
is ACL/EMNLP/NAACL/LREC-family (ARR-governed).

## Phase 3.6 — DOCX / Word path (only when the venue requires it)

Marketplace `officecli-academic-paper` capability, house form ([H1584 (Opus 4.8) — Uprava skill-consolidation/memory/academic-writing wave](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1584-Opus_Uprava_skill-consolidation-memory-academic-wave_24.07.26.md)): most of this org's
venues take PDF/MD; **do not** emit DOCX by default.

When author guidelines require `.docx` (APA/Chicago/IEEE/MLA Word upload):

1. Prefer Pandoc from the camera-ready Markdown/MDX:
   `pandoc manuscript.md -o <id>_submission.docx --citeproc` (with the paper's `.bib` if any).
2. If equations break, fall back to LibreOffice headless on an intermediate ODT, or a
   human-edited Word pass — note which path was used in the checklist.
3. Never round-trip a macro-heavy workbook through openpyxl as a "paper export"
   (skill-fit audit: analyzing-spreadsheets trial — lossy on near-max-row `.xlsm`).
4. Ship `<id>_submission.docx` next to the pack; leave the Markdown source canonical.

## Phase 4 — Bookkeeping

- Bump readiness/status in `Uprava/ARTICLES.md` per the `/articles-update` discipline (stable ID, never renumber) and regenerate the dashboard (`python Uprava/tools/build_dashboard_data.py`).
- The actual submission is a human action: add a @DO to `Uprava/GTD_NEXT_ACTIONS.md` with a link to the pack.
- Commit both repos.

Report: pack file links, failed-checklist items, new readiness score.

_Dr. Mārcis Gasūns_
