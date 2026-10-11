---
name: "commentary-strategies-editorial-pipeline"
description: 'Synchronize and validate CommentaryStrategies publication state — manuscript frontmatter, ARTICLES/GTD/roadmap status, readiness gates, ORCID/byline metadata, validate.py outcomes. Triggers: align A## readiness across trackers, make validate.py green.'
---

# commentary-strategies-editorial-pipeline

Use this skill when the user asks to run the migrated source command `commentary-strategies-editorial-pipeline`.

## Command Template


# /commentary-strategies-editorial-pipeline

_Created: 11-07-2026 · Last updated: 25-08-2026_

Working dir: [`CommentaryStrategies`](https://github.com/gasyoun/CommentaryStrategies).

**Core rule:** publication state is cross-file *evidence*, not a single-file edit. Before changing status, readiness, author metadata, or roadmap text, identify the article/A-number and verify **every** canonical tracking point that mentions it. Apply only the smallest consistency fix — never invent a readiness stage or close a gate without evidence.

## Canonical inputs

- `CLAUDE.md` — repo rules, corpus-integrity commands, forbidden strings, source/generated boundaries.
- `.ai_state.md` — current publication queue, completed milestones, open gates, recent cross-check decisions.
- Article manuscripts + cover/readiness files under `articles/`.
- Hub files: `ARTICLES`, GTD, `PUBLICATION_ROADMAP_2026Q3.md` (in-workspace or sibling planning repos).
- **Active-workstream state (post-01-07-2026), so status claims stay honest:** the Sundara apparatus now has a Phase-2 commentator layer — licensed Gita Supersite commentaries (rights CLEARED CC BY 4.0, `data/valmiki_PERMISSION.md`), Leonov/Kostina's own **1,058 notes** as dedup baseline (`data/leonov_own_notes.json`), edition comparison critical↔southern (`data/edition_comparison/`), hybrid DeepSeek/agent generation, and the **register gate** (MG/Kostina) that blocks any scale-up: an interactive voting sheet emitted by the shared `render_review_sheet(items, config)` ([`csl-pyutil`](https://github.com/sanskrit-lexicon/csl-pyutil) v0.3.0/v0.3.1, the [/review-sheet](https://github.com/gasyoun/claude-config/blob/main/commands/review-sheet.md) V1–V8 standard), named `commentarystrategies-sundara-apparatus_<scope>_review.html` + `..._decisions.json` — a generic `review.html`/`decisions.json` is banned, as is hand-rolled sheet HTML; `PILOT_REVIEW.md` is the read-only fallback. Pipeline skill: [`/commentary-apparatus-ingest`](https://github.com/gasyoun/claude-config/blob/main/commands/commentary-apparatus-ingest.md). A roadmap/readiness row claiming Phase-2 is "blocked on OCR" is stale — the corpus is local and licensed.

## Cross-check targets

- Readiness must use the current `N/5 - <stage>` vocabulary from the hub, not ad-hoc draft labels.
- ORCID + email match the canonical author source (usually `Uprava/AUTHOR.md` when cited in `.ai_state.md`).
- Byline form may differ by journal — treat Cyrillic-vs-Latin as an explicit **gate**, not an automatic correction.
- Roadmap and GTD mentions must not contradict the source manuscript/readiness files.
- `.ai_state.md` records the validation status and the exact discrepancy closed or left open.

## Hard rules

- **Never** add the false Leonov attribution `М.: Наука, 2022`; use the repo-approved ongoing translation/editorial wording. Known pre-existing violators to clean, not imitate: `scripts/sundara_ch1_enrich.py` + `data/sundara_ch1_corpus_relevance.json`.
- Preserve the correct oblique form `Парибка`.
- Do **not** add a fifth analytical axis without explicit user permission.
- Commentators may be named in the apparatus, **never in an article title** (hard-rule #4); Kazansky's name likewise stays out of titles.
- Do not treat generated web-asset dumps or archives as primary source unless the task explicitly targets them.
- **Rejected/QA-removal logs are evidence, never destroy them** — including Phase-2 reject files with their reason buckets.

## Workflow

1. Inspect `CLAUDE.md`, `.ai_state.md`, and the relevant article/readiness files before deciding what state is canonical.
2. Resolve the target A-number, manuscript path, cover/readiness files, and hub rows in `ARTICLES`, GTD, roadmap.
3. Cross-check readiness number, status wording, ORCID, email, byline form, and open gates across every tracking point.
4. Apply the smallest consistency fix needed.
5. Run validation after content edits and record the command/result in `.ai_state.md` or handoff notes:
   ```sh
   python scripts/validate.py
   python scripts/derive_urn.py --check     # when corpus records or generated TEI/pages are touched
   ```
   If validation flags pre-existing unrelated issues, record them separately — do not hide them with unrelated edits. When data changed, also account for **generated-artifact reproducibility**: every generator (`export_tei.py`, `build_pages.py`, the enriched HTML/stats builders) must reproduce its artifact with no git diff — that IS the CI corpus gate; note any refresh still owed. Report HTML must carry the design-system markers (`css/commentary.css` link + literal `<main class="container">`) or `validate.py` fails — **emitter-generated `*_review.html` voting sheets are exempt by construction** (self-contained inline CSS+JS, opened from `file://`): exclude them from that check instead of hand-editing emitter output.
6. If stopping before edits, leave the exact discrepancies + candidate files for the next agent.

## Concurrency / multi-actor environment

- A **separate-account automation merges PRs to `main` and switches branches mid-session** — verify end-state on `origin/main` directly before reporting a status as landed; prefer branch+PR over direct main; isolate contended work in a fresh `git worktree`.
- The host cycles long agents: batch work as small resumable write-only steps (no git inside subagents), commit centrally.

## Stop conditions

Stop and report when canonical state conflicts across sources, an article can't be mapped to an A-number, a gate depends on author/editor sign-off, or validation fails for reasons outside the requested change.

_Mirrors the Codex skill `commentary-strategies-editorial-pipeline` (kept in sync via [`/claude-codex-skill-porter`](https://github.com/gasyoun/claude-config/blob/main/commands/claude-codex-skill-porter.md))._

_Uplifted (Phase-2 reality · validate robustness · concurrency) by Fable 5 (`claude-fable-5`), 09-07-2026, H444 (Fable 5) - CommentaryStrategies commentary-apparatus-ingest skill._

_Dr. Mārcis Gasūns_
