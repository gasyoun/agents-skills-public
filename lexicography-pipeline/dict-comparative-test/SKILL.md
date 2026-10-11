---
name: "dict-comparative-test"
description: 'Run ONE falsifiable comparative test between two dictionaries - omission, sense-order, or fitted-index clause - against an independent control + shuffled null. Triggers: "test whether MW copied PWG", "fitted-index census". Opus 4.8 (claude-opus-4-8) tier.'

---

# dict-comparative-test

Use this skill when the user asks to run the migrated source command `dict-comparative-test`.

## Command Template


# /dict-comparative-test

_Created: 17-07-2026 · Last updated: 25-08-2026_

The csl-atlas forensic-suite discipline (H761 H796 H801), generalized: a claim that one dictionary DESCENDS FROM another is not one question but a bundle of separable clauses — shared omissions, shared errors, false citations, copied sense-order, fitted citation loci — and each clause gets its own test, its own control, and its own verdict. The clauses never collapse into a single "is it plagiarism" answer: Böhtlingk's 1883 charge against Monier-Williams measures ≈8× coupling on omissions, null on errors, and null on citations — descent on the inventory side, independence on the apparatus side. A skill that returns one number for "copying" has destroyed the finding.

Argument: the clause to test + the two dictionaries (+ the article it folds into). Resolve the F-suite location from the repo's existing `scripts/forensic/` if one exists.

Evidence base: H761 H796 H801 (3 sessions, csl-atlas). Dedupe: `/crosswalk-build` builds the alignment asset, it does not test a claim against it; `/entry-anatomy` describes one entry's structure; `/grammar-claims-audit` audits a textbook's claims about the LANGUAGE — this skill owns falsifiable claims about the RELATIONSHIP BETWEEN TWO DICTIONARIES. **Ghost-word / ghost-stock / coverage-saturation inventory forensics** (H1310/H1323/H1575/H1576) → `/ghost-stock-audit`, not a fitted-index or sense-order clause here.

## Phase 0 — Read-only invariant + safety gate

**A comparative test is READ-ONLY by design. It consumes dictionary data and emits new files; it never regenerates, reseeds, or refreshes an existing data asset.** State this invariant in the deliverable. It is not a stylistic preference — it is the gate that keeps this genre away from the csl-atlas footgun below.

**HARD SAFETY GATE — csl-atlas seeders destroy human review** (`project_csl_atlas_refresh_scope` in org memory + the csl-atlas repo memory store). If a test seems to want "fresh inputs", do NOT reach for a refresh-all:

- **Safe to re-run** (idempotent against inputs): `build-dict-comparison`, `build-citation-apparatus`, `build-sense-depth`, `build-semantic-fields`, `build-structural-register`, `build-xref-lineage`, `build-three-axis-comparison`, `build-coverage`, `build-mw-depth`.
- **NEVER include in a refresh:** `build-r2-checkpoint-review`, `build-r2-drift-explanation`, `build-r2-promotion-experiment`. The checkpoint builder is by design a *seeder* — it overwrites `src/data/review/r2-checkpoint-review.json` with empty `needs-review` skeletons, wiping a committed human sign-off, and the promotion packet then cascades to "not reviewed-ok; skipped" for every row. That is a regression, not a refresh. (PR #82 made the checkpoint preserve-by-default; even so, never blind-include the derived R2 packets — check their diffs.)
- This skill needs none of them. If you believe you need a seeder, you have left the genre — stop and surface it.

Contention: csl-atlas is a guarded repo (`.githooks/pre-commit` blocks main-tree commits). Work in a worktree: `git worktree add -b <branch> ../csl-atlas-<slug>-$$ origin/main` (the `-$$` session token, H1336).

## Phase 1 — Isolate ONE clause, and say what would falsify it

- **One clause, one test.** Name the clause in the source's own words where a documentary source exists — Müller's itemization of Böhtlingk's charge (11 June 1881, Stache-Weiske 2015) gives four literal clauses; *"die Reihenfolge der Bedeutungen einfach abgeschrieben"* is a different claim from "shared errors", and F5 citation-order only *proxies* it. Proxy ≠ clause: record which is which.
- Write the formalization before the code: what measurement, at what threshold, against what control, would make this clause FALSE.
- Check which clauses the target article already measures — the value of a new test is the gap it closes, and an article that measures three of four clauses names its own next task.

## Phase 2 — Reuse the F-suite, do NOT rebuild

The forensic scripts are a numbered suite under `scripts/forensic/` — `f5_entry_comparison.py` (citation-order), `f8_mbh_census.py` / `f8_mbh_verify.py` (fitted index), `f9_shared_omission.py` (omissions), `f10_sense_order.py` (sense order). Read the nearest sibling and adapt it; take the next free `fN`. `parse_cslorig.py` already segments entries — extend it, never re-write an entry parser. Reuse the offline MT cache (`_f6_tcache.json`) for cross-lingual work; **never a live LLM in the measurement path** — the numbers must be recomputable by a third party.

Anchor the comparison on a real, defensible set (F9's anchor: SKD ∩ VCP = 6,941 indigenous words). State the anchor and its size in the output.

## Phase 3 — The control is mandatory, not optional

**Without an independent control, "copying" is indistinguishable from shared lexicographic logic.** Senses have a partly-forced canonical order (literal → figurative → technical); two independent dictionaries of the same language share omissions because the same words are rare. So every clause needs both:

1. **An independent third dictionary** — Apte is the canonical control for the MW/PWG pair (English senses, demonstrably independent). A-vs-B concordance must EXCEED A-vs-control to count as descent.
2. **A shuffled null** — the same statistic over randomized order/inventory, to fix what chance looks like.

Report the differential, not the raw number: F9's finding is not "MW gap-sensitivity 12.3×", it is "12.3× vs independent Apte 1.5× — an ≈8× differential". A raw coupling number with no control is not a result and must not be published as one.

## Phase 4 — The circularity gate (a failed gate STOPS the test)

Where the test depends on a fitted mapping (the f8 fitted-index shape: fit cumulative adhyāya offsets on PWG anchors → map `MBH. 7,N` to `(adhyaya, shloka)`), the fit must be **held out** on a second source (fit on PWG, hold out on MW).

**If the held-out gate fails, the downstream error/classification test MUST NOT RUN.** Report the failure and stop. Running a classification on top of a fit that failed its own held-out check measures the fit, not the claim — the result would be circular and worthless. This is a hard stop, not a caveat to note in passing.

## Phase 5 — A null is a result; publish it as one

Ruling a clause is a two-sided outcome and both sides are deliverable:

- **Positive** closes the clause and upgrades the article (F9: descent confirmed from the negative-space side).
- **Null is equally publishable** — "sense order is convergent, not copied", "the error clause is null" — and still fills the gap in the charge. A failed vulgate alignment is a real, publishable negative about the edition, not a failed session.
- **Refuse the overclaim.** F9 found ≈8× coupling AND that MW independently supplies 54.6% of PWG's blind spots → *descent, not blindness*. Report both halves; the honest verdict is usually narrower than the charge.

## Phase 6 — Deliver, fold, log

- **Artifacts:** `scripts/forensic/fN_<clause>.py` + `data/forensic/<CLAUSE>_TEST.md` (+ meta) + a **numbers-only CSV**. Every count recomputable.
- **Fold into the EXISTING article, don't mint a new one.** A new clause is a section in the paper that owns the charge (A10 `article_21_apparatus_not_errors.md` — §3.5 F9, §3.6 F10), plus a row in its method table, plus abstract + reproducibility + references. No new `ARTICLES.md` id for a clause; bump the existing row's readiness instead.
- **Log the caveat, not just the number.** The non-independence of PWG/PW/MW is `SanskritLexicography/FINDINGS.md` §83 — a comparative result that ignores it will be misread. `/findings-append` any new one.
- Roadmap sweep: if the clause was recorded BLOCKED anywhere, flip that note with the PR link — H761's whole premise was that the roadmap's "no free bulk vulgate exists" had gone false.
- PR + auto-merge from the worktree; `/handoff-close`; model provenance with tier AND exact version.

_Dr. Mārcis Gasūns_

## Gotchas

- **Recension before arithmetic.** PWG/MW cite the Nīlakaṇṭha/Bombay vulgate; BORI-critical texts (GRETIL/DCS) are a structural dead end for locus arithmetic (`SanskritLexicography/DEAD_ENDS.md` §8/§8b). Testing a citation clause against the wrong recension measures the recension gap, not the claim.
- **Rights-gated bulk text stays gitignored.** The 58.9 MB `nilakantha_vulgate_full.jsonl` is present locally, never committed — commit the census and the numbers-only CSV, not the third-party text.
- **A digitization count is not an edition match.** sanatana.in's 83,971 shlokas were never verified against a specific print vulgate; the fitted-index + held-out gate is exactly what tests that. Don't assume the alignment holds because the scrape succeeded.
- **Proxy drift.** F5 citation-order (0.811) was read as the sense-order clause for months. When a proxy stands in for a clause, say so in the article, or the clause silently counts as measured.

_Dr. Mārcis Gasūns_
