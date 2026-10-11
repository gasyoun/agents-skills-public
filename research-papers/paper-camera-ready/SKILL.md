---
name: "paper-camera-ready"
description: 'Post-acceptance camera-ready pass for an accepted Uprava/ARTICLES.md paper — LOCK metadata (ORCID, DOI), verify figures+cites, readiness to 5, GTD submit-action. Triggers: "camera-ready Axx". Fable 5 (claude-fable-5), Opus 4.8 (claude-opus-4-8) fallback. Do not use before a venue has accepted the paper — for pre-submission review use /paper-referee, for revision responses use /paper-revision-response.'
---

# paper-camera-ready

Use this skill when the user asks to run the migrated source command `paper-camera-ready`.

## Command Template


# /paper-camera-ready <Axx>

_Created: 14-07-2026 · Last updated: 05-09-2026_

The paper family stops at submission: [`/paper-scaffold`](https://github.com/gasyoun/claude-config/blob/main/commands/paper-scaffold.md) → [`/paper-author-pass`](https://github.com/gasyoun/claude-config/blob/main/commands/paper-author-pass.md) → [`/paper-referee`](https://github.com/gasyoun/claude-config/blob/main/commands/paper-referee.md) → [`/paper-revision-response`](https://github.com/gasyoun/claude-config/blob/main/commands/paper-revision-response.md) → [`/paper-submission-pack`](https://github.com/gasyoun/claude-config/blob/main/commands/paper-submission-pack.md). Nothing handles what happens **after a venue accepts** — reformatting to the camera-ready template, de-anonymizing, locking metadata, and producing the final files the human uploads. That is this skill. Where `/paper-submission-pack` builds the package that *goes into review at readiness 4–5*, this pass takes an *accepted* paper to the final camera-ready deliverable and 5/5.

Argument: the stable paper ID (`Axx` / `Mxx`) from [`Uprava/ARTICLES.md`](https://github.com/gasyoun/Uprava/blob/main/ARTICLES.md), or enough title to resolve one.

## Phase 0 — Precondition: acceptance is real

This pass fires **only post-acceptance**. Confirm the paper has an actual accept decision (acceptance email/notification, venue portal status, or an `MG @DO` in GTD recording it) before touching anything. If the paper is still pre-decision — under review, revising, or never submitted — **stop** and route to [`/paper-submission-pack`](https://github.com/gasyoun/claude-config/blob/main/commands/paper-submission-pack.md) (or `/paper-revision-response` for a revise-and-resubmit). A camera-ready pass on an un-accepted paper is wrong work.

## Phase 1 — Resolve the paper

Look it up in [`Uprava/ARTICLES.md`](https://github.com/gasyoun/Uprava/blob/main/ARTICLES.md): repo, manuscript path, current readiness, the venue that accepted it. Read the FULL accepted manuscript. Fetch the venue's **camera-ready / final-version author instructions** (distinct from the submission guidelines — camera-ready has its own template, page allowance that often grows by a page, and copyright step); cite the version/date fetched.

## Phase 2 — Apply the venue's official template

Reformat to the venue's camera-ready style: the required LaTeX document class / Word template, the final page/word limit (usually the accepted length + any extra allowed page), section numbering and reference style. **Remove anonymization** — the review copy's blinded author block, self-citations written in the third person, and any anonymized-repo placeholders (`[anonymous]`, redacted URLs) become the real byline, real self-citations, and real full blob URLs. Confirm nothing anonymized survives (a leftover "the authors' prior work [anonymous]" is a camera-ready defect).

## Phase 3 — Final copyedit pass

A single careful copyedit — typography, hyphenation, consistent terminology, caption/label agreement, orphaned references, table/figure numbering in reading order. This is the last edit the paper gets before it is frozen, so it is thorough but **substance-preserving**: numbers, claims, and citations do not change here (any genuine substantive doubt found this late is a HOLD finding for the human, not a silent fix — the paper is already accepted on its current substance).

## Phase 4 — LOCK the metadata (the load-bearing step)

Freeze every publication-identity field from the single sources of truth — **never ask the user, never invent**:

- **Byline / ORCID / affiliation / email** — from [`Uprava/AUTHOR.md`](https://github.com/gasyoun/Uprava/blob/main/AUTHOR.md) (the canonical source; the paper byline carries no "Dr." and is distinct from the file-authorship line). Apply the venue's required author-block format exactly.
- **Copyright / license form** — the venue's copyright transfer or CC-license election (ACL Anthology → the ACL license; many DH venues → CC BY). Record which was chosen; the signed form itself is a human action (Phase 6).
- **Funding / acknowledgements** — final text, from `AUTHOR.md` / the repo's funding note; empty is a deliberate choice, not an oversight — state it.
- **DOI slot** — reserve/record the venue or Zenodo DOI slot. Do NOT fabricate a DOI. Heed the A-series trap logged in [`Uprava/ARTICLES.md`](https://github.com/gasyoun/Uprava/blob/main/ARTICLES.md): a DOI string that resolves to an unrelated preprint is worse than a blank — leave it a labelled placeholder and mirror the mint to GTD if unminted.
- **`CITATION.cff`** — update to the published version, date, DOI, and ORCID byline (pairs with [`/cut-release`](https://github.com/gasyoun/claude-config/blob/main/commands/cut-release.md) if the repo is citable). Validate with `cffconvert --validate` if available.

## Phase 5 — Verify every figure, table, and citation renders camera-ready

- Every figure/table renders at final resolution and fits the column/page — regenerate from the committed source if the review copy used a draft render.
- Every in-text citation resolves to a complete, correctly-styled bibliography entry (no `?`, no broken `\cite`, no dangling DOIs).
- Cross-references (section/figure/table/equation numbers) all resolve after reformatting.

A broken cite or a figure that overflows the camera-ready column is a blocker, reported as top remaining-work — not a note.

## Phase 6 — Bookkeeping

- Deliver the reformatted manuscript + locked metadata as a branch + PR in the owning repo, auto-merge per the large-work rule (never commit to the default branch from an unattended run).
- Bump readiness to **5** and set status to camera-ready/accepted in [`Uprava/ARTICLES.md`](https://github.com/gasyoun/Uprava/blob/main/ARTICLES.md) per the [`/articles-update`](https://github.com/gasyoun/claude-config/blob/main/commands/articles-update.md) discipline (stable ID, never renumber), then regenerate the dashboard: `python Uprava/tools/build_dashboard_data.py`.
- The final upload is a human action: add an `MG @DO` row to [`Uprava/GTD_NEXT_ACTIONS.md`](https://github.com/gasyoun/Uprava/blob/main/GTD_NEXT_ACTIONS.md) — "submit camera-ready files for `<Axx>` to `<venue>` by `<deadline>`" — linking the PR and listing any human-only sub-actions (sign the copyright form, mint the DOI).
- Commit both repos.

Report: the camera-ready PR link, the locked-metadata summary, any HOLD/verify items left for the human, and the new readiness (5).

## When NOT to use

- **Pre-acceptance** — the paper is under review, revising, or never submitted → [`/paper-submission-pack`](https://github.com/gasyoun/claude-config/blob/main/commands/paper-submission-pack.md) (packaging) or [`/paper-revision-response`](https://github.com/gasyoun/claude-config/blob/main/commands/paper-revision-response.md) (revise-and-resubmit).
- **Substance still in doubt** — if a figure or claim was never verified against committed data, that is a `/paper-referee` job; a camera-ready pass polishes an already-settled paper, it does not re-review it.
- **A repo release/tag** (not a paper) → [`/cut-release`](https://github.com/gasyoun/claude-config/blob/main/commands/cut-release.md).

## Safety notes

- **Never fabricate a DOI or a metadata field.** All identity fields come from [`Uprava/AUTHOR.md`](https://github.com/gasyoun/Uprava/blob/main/AUTHOR.md); an unminted DOI stays a labelled placeholder with a GTD mint action, never a made-up string (the unrelated-preprint DOI incident in ARTICLES.md is the cautionary case).
- **Substance is frozen.** Post-acceptance, numbers/claims/citations do not change under "copyedit" — a late substantive doubt is a human HOLD, not a silent edit; the venue accepted the paper on its current substance.
- **De-anonymization is complete or it is a defect.** Confirm no blinded byline, third-person self-citation, or `[anonymous]` placeholder survives into the final files.
- **Register-sensitive** — byline, acknowledgements, and cover text are the author's public voice; run at the Fable 5 (`claude-fable-5`) tier, Opus 4.8 (`claude-opus-4-8`) fallback.

_Dr. Mārcis Gasūns_
