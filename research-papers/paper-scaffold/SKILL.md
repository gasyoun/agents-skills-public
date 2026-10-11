---
name: "paper-scaffold"
description: 'Scaffold a new paper idea to a readiness-2 skeleton — Axx ID from ARTICLES.md, home-repo choice, outline with data-inventory + literature matrix stub, venue line. Triggers: new paper on X, scaffold the paper. Do not use on an existing paper — this creates a brand-new Axx skeleton; for revising an existing draft use /paper-author-pass or /paper-referee.'

---

# paper-scaffold

Use this skill when the user asks to run the migrated source command `paper-scaffold`.

## Command Template


# /paper-scaffold

_Created: 03-07-2026 · Last updated: 05-09-2026_

`/articles-update` registers a paper; nothing scaffolds one. With a 15-paper roadmap, ideas surface faster than skeletons get built — and the skeletons that work here are data-first: the papers that reach 5/5 stand on committed artifacts, so the scaffold's job is to force the data inventory BEFORE the prose.

Argument: working title or one-line idea.

## Phase 1 — Prior art on the idea itself

Check `Uprava/ARTICLES.md` end-to-end first: an existing paper may already own this idea (then the move is a note on ITS row, not a new ID), and its status/blocker columns may show it parked deliberately. A duplicate registration is worse than no registration.

## Phase 2 — Register

Per the `/articles-update` discipline: take the "Next free ID" marker's value (`Axx`), bump the marker, add the row (readiness 1 or 2, status, blocker = what data is missing), grouped under its repo. IDs are append-only, never reused.

## Phase 3 — Home repo + skeleton

Home repo per the data-locations split, stated here in full because there is no `data-locations` rule file to look up (the estate's only copy is a `reference` memory in a Windows-path store, unloadable on a mac session): Cologne-dict **metadata** and **all non-Cologne** dictionary data → [SanskritLexicography](https://github.com/gasyoun/SanskritLexicography); a Cologne dictionary's own raw source text (`wil.txt`, `mws.xml`, `pwg.txt`) stays in its Cologne repo (`csl-orig`, `pwgxml`, `mwsxml`) and is never copied into a paper repo; repo-specific findings → that repo's own `papers/` or `docs/articles/` convention — follow where its siblings live. Create `papers/<Axx>_<slug>.md` (or the repo's pattern) with dated header + byline and these sections:

- **Claim** — the one-sentence finding the paper will defend (if it can't be stated yet, readiness is 1 and the skeleton says so honestly).
- **Data inventory** — a table: each intended result → the committed artifact that backs it (full blob URL) → its status (exists / needs deriving / needs a `/data-release`). Gaps in this table ARE the work plan; a row with no artifact is a flagged blocker, not a hidden one.
- **Outline** — section stubs with one line each of intended content; no filler prose.
- **Comparanda / literature** — the 2–3 obvious analogs a referee would name, found now, not at review time. Prefer seeding via `/paper-search "<claim keywords>"` and asking it for the literature matrix — `--matrix` is a **session-side deliverable, not a script flag** (`paper_search.py` implements none; verified 26-08-2026, see [`/paper-search`](https://github.com/gasyoun/claude-config/blob/main/commands/paper-search.md) § Run) — then paste the keep=Y rows here (title + one-line method + relevance). Philology-only papers still need manual JSTOR/Brill notes — arXiv/S2 will not cover them.
- **Ablation / isolation plan** (method / detector / pipeline papers only; `n/a` for pure philology) — table: `factor | what we hold fixed | what we vary | metric | expected direction | committed artifact`. Forces the marketplace `ablation-planner` discipline without a third-party skill. If the paper claims a component "helps", this table must name the remove-one / swap-one test.
- **Venue candidates** — one line; a serious shortlist is `/venue-scout`'s job later.
- **Provenance** — scaffolding model tier + exact version, date.

## Phase 4 — Wire + deliver

Regenerate the dashboard (`python Uprava/tools/build_dashboard_data.py`), add a GTD row only if a human action gates the next step (data access, a rights question) — an idea skeleton alone is agent-resumable and needs no @DO. Commit both repos per their rules. Report: new ID, skeleton link, the data-inventory gaps as the de-facto next steps.

_Dr. Mārcis Gasūns_
