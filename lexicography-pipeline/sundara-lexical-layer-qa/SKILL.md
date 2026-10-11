---
name: "sundara-lexical-layer-qa"
description: 'Maintain Sundara lexical/etymology/cross-text/commentator layers in CommentaryStrategies — QA density targets, gate behavior, lemma dedup vs Leonov/Kostina''s 1058 notes, counts. Trigger: "QA Sundara ch.N notes".'

---

# sundara-lexical-layer-qa

Use this skill when the user asks to run the migrated source command `sundara-lexical-layer-qa`.

## Command Template


# /sundara-lexical-layer-qa

_Created: 04-07-2026 · Last updated: 27-08-2026_

Working dir: [`CommentaryStrategies`](https://github.com/gasyoun/CommentaryStrategies), Sundarakanda annotation work.

**Core rule:** preserve the quality gate. Accepted lexical notes must be non-obvious, evidence-backed, deduplicated at the intended scope, and marked for human review where the layer requires it. **Rejected candidates are evidence, not trash** — never destroy the rejected/QA-removal logs (this includes Phase-2 reject files with their reason buckets).

## Source files

- `CLAUDE.md` — layer definitions, target density, validation commands, hard rules.
- `SUNDARA_COMMENTARY_RATIONALE.md` — decision-ledger rationale and acceptance regimes.
- `.ai_state.md` — latest runs, open follow-ups, generated-output refresh needs.
- `data/lexical/ch{N}.json`, `ch{N}.rejected.json`, `ch{N}.qa_removed.json`.
- `data/sundara_ch{N}_commentary_to_add.json` and the book-level merged files (`data/sundara_book_stats.json`) for final totals.
- **Phase-2 layer (post-01-07-2026):** [`data/leonov_own_notes.json`](https://github.com/gasyoun/CommentaryStrategies/blob/main/data/leonov_own_notes.json) — Leonov/Kostina's OWN **1,058 notes** (model II tier-1, the ~36% benchmark, **primary dedup baseline** for commentator notes); the licensed Gita Supersite commentaries `data/valmiki_commentaries/kanda_5_sundarakanda/` (tilaka/bhūṣaṇa/śiromaṇi, rights CLEARED CC BY 4.0); `data/analysis/phase2_pilot/` candidates + the register gate — an interactive voting sheet emitted by the shared `render_review_sheet(items, config)` ([`csl-pyutil`](https://github.com/sanskrit-lexicon/csl-pyutil) v0.3.0/v0.3.1, the /review-sheet V1–V8 standard), `commentarystrategies-sundara-apparatus_<scope>_review.html` + `..._decisions.json` (generic `review.html`/`decisions.json` and hand-rolled sheet HTML are banned; `PILOT_REVIEW.md` is the read-only fallback); `data/edition_comparison/` (critical↔southern — the apparatus is over the **southern vulgate** Leonov translates).

## Acceptance regimes (don't mix criteria between layers)

- **Base notes** close real gaps in the Russian prose/commentary apparatus.
- **Lexical/etymological notes** may gloss non-obvious terms, compounds, and etymologies even when the prose translation is clear.
- **Cross-text notes** are softer evidence — keep `review_required:true`.
- **Commentator (Phase-2) notes** report what Tilaka/Bhūṣaṇa/Śiromaṇi actually say; deduped against the подстрочник AND Leonov/Kostina's own 1,058 notes AND Phase-1; built via [`/commentary-apparatus-ingest`](https://github.com/gasyoun/claude-config/blob/main/commands/commentary-apparatus-ingest.md) (segmentation → hybrid DeepSeek/agent drafting → merge → **register gate**: the emitter-generated `commentarystrategies-sundara-apparatus_<scope>_review.html` sheet on the /review-sheet standard, three verbs approve · reject · defer with the rewrite request going in the note — nothing scales past the pilot without MG/Kostina approval).

## Density ladder

Base ~5% → +lexical ~24% (Grintser target) → Phase-2 commentator layer toward **~36% (Leonov parity**, 1058/2859). A recap chapter legitimately sits far below target (vocabulary introduced earlier, deduped out); Phase-2 accept rates are inherently low (~6% in the pilot — two-thirds of commentary restates the подстрочник).

## Adversarial gate

Reject transparent participles, standard epithets, obvious compound sums, in-chapter root duplicates, and notes already covered by an earlier first occurrence; for commentator notes also reject restates-подстрочник, duplicates of a Leonov/Kostina own note, and pure Sanskrit grammar invisible to a Russian reader. **A high rejection rate is a quality signal, not a failure.** Check at least: accepted-note rationale · rejected-candidate reason · chapter density · global lemma-dedup impact · whether recap chapters are low because vocabulary was introduced earlier · whether a commentator note overlaps `data/leonov_own_notes.json`.

## Invariants

- Preserve rejected logs and QA-removal logs (Phase-1 and Phase-2 alike).
- Preserve `review_required:true` on lexical/cross-text/commentator notes unless a human editor explicitly resolves that review.
- Do not treat low density in a recap chapter as a gap without first checking first-occurrence dedup.
- Do not change Leonov text while adjusting notes unless explicitly requested.
- **False-attribution guard:** never «М.: Наука 2022» for Leonov (`validate.py` enforces); known pre-existing violators to clean, not imitate: `scripts/sundara_ch1_enrich.py` + `data/sundara_ch1_corpus_relevance.json`.

## Workflow

1. Inspect the source files above and the target `data/lexical/` chapter files (plus `data/analysis/phase2_pilot/` when QAing commentator notes).
2. Identify the layer (base / lexical-etymological / cross-text / commentator); keep acceptance criteria separate.
3. Check accepted notes against the adversarial gate and the rejected logs before adding/removing/restoring; for commentator notes, cross-check `data/leonov_own_notes.json` overlap.
4. Preserve `review_required:true` where the layer is marked for human editorial review.
5. Recompute or report density + dedup implications when changing chapter or book totals.
6. Run `python scripts/validate.py` **and, when corpus records or generated TEI/pages are touched, `python scripts/derive_urn.py --check`** after data/content edits; record any generated corpus/HTML refresh still needed (enriched HTML, `data/sundara_book_stats.json`, corpus summary counts — CI fails if a generator leaves a git diff).

## Concurrency / multi-actor environment

- The host cycles long agents: prefer **many small parallel write-only agents** (~2–3 chapters each, no git in-agent — avoids index-lock races), resumable by skipping existing files; merge centrally with deterministic Python.
- A **separate-account automation merges PRs to `main` and switches branches mid-session** — verify end-state on `origin/main` directly, prefer branch+PR over direct main, and isolate contended work in a fresh `git worktree`.

## Episodic Pāṇini cross-ref

For the rare verse whose note turns on a Pāṇinian grammatical crux, consult [`/panini-commentary-corpus`](https://github.com/gasyoun/claude-config/blob/main/commands/panini-commentary-corpus.md) for the commentator frame. **Episodic only** — Sundara is a «Литературные памятники» edition, not a grammar-teaching edition; Pāṇinian commentary is not a standing layer.

## Reporting

Report: chapter number · accepted/rejected counts · density · notes added/removed/restored · dedup rationale (incl. Leonov-own-notes overlap for commentator notes) · validation command + result · generated outputs still needing refresh (enriched HTML, corpus summary counts).

## Stop conditions

Leave a research note when evidence is insufficient, a low-yield recap chapter may be correct by design, or a change would remove rejected-candidate audit evidence. A note needing traditional-commentary evidence is **no longer blocked on OCR** — the licensed Gita Supersite corpus is local and segmentable; route it through [`/commentary-apparatus-ingest`](https://github.com/gasyoun/claude-config/blob/main/commands/commentary-apparatus-ingest.md) instead (still gated on the pilot-register approval for anything beyond the pilot).

Canonical source: `~/.claude/commands/sundara-lexical-layer-qa.md` (this repo); Codex and Grok twins are GENERATED from it by `scripts/generate_playbook_twins.py sundara-lexical-layer-qa` and must never be hand-edited.

_Uplifted (Phase-2 reality · validate robustness · concurrency) by Fable 5 (`claude-fable-5`), 09-07-2026, H444._

_Dr. Mārcis Gasūns_
