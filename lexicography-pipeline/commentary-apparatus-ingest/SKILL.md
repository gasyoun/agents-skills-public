---
name: "commentary-apparatus-ingest"
description: 'Run/resume Phase-2 commentator apparatus for Leonov Sundarakāṇḍa in CommentaryStrategies — segment Gita Supersite commentaries per verse, dedupe vs Leonov/Kostina''s 1058. NEVER past pilot without MG/Kostina approval. Trigger: "run Phase 2".'

---

# commentary-apparatus-ingest

Use this skill when the user asks to run the migrated source command `commentary-apparatus-ingest`.

## Command Template


# /commentary-apparatus-ingest

_Created: 11-07-2026 · Last updated: 25-08-2026_

Working dir: [`CommentaryStrategies`](https://github.com/gasyoun/CommentaryStrategies). Canonical method
manual: [`docs/PHASE2_METHOD.md`](https://github.com/gasyoun/CommentaryStrategies/blob/main/docs/PHASE2_METHOD.md)
(+ [`docs/PHASE2_SUNDARA_HANDOFF.md`](https://github.com/gasyoun/CommentaryStrategies/blob/main/docs/PHASE2_SUNDARA_HANDOFF.md)) —
this skill orchestrates that pipeline; it does not replace reading it on first contact.

**Core rule:** the pipeline is **deterministic at the edges, LLM only in the middle, human at the gate**.
Segmentation and merge are reproducible Python; only candidate drafting/judging is model work; nothing
reaches `data/sundara_ch{NN}_commentary_to_add.json` without the MG/Kostina interactive review. **Never
scale past the pilot until the register is approved.**

**Prior art:** every driver already exists in `scripts/` — run `/prior-art` before writing ANY new
Python. Segmenter `extract_yellow_sargas.py` · Leonov baseline `extract_leonov_notes.py` · merge
`merge_phase2_pilot.py` · apply `apply_phase2_decisions.py` · edition diff `compare_editions.py`. Write a
new script only for a proven gap, and register it. **Exception — the review page:**
`build_pilot_review_html.py` predates the shared emitter and its inline HTML template is superseded by
`render_review_sheet()` (see STEP 4); port that script onto the emitter, never copy its markup.

## Arguments

Sarga numbers (e.g. `35 36 37`), a coverage tier (`yellow` = the 14 🟡 commentary-rich/Leonov-thin
sargas from `data/analysis/sundara_coverage.json`), or `resume` (skip sargas whose
`sarga_NN_candidates.json` already exists).

## Inputs (all local, rights CLEARED — CC BY 4.0, Gita Supersite permission)

- Commentaries: `data/valmiki_commentaries/kanda_5_sundarakanda/{tilaka,bhusana,siromani}_sarga_NN.txt`
  (Devanagari, verse markers `।। 5.<sarga>.<verse> ।।`; coverage tilaka 66 / bhūṣaṇa 68 / śiromaṇi 65 of 68;
  tattvadipika only sargas 1–6; kataka/dharmakutam/tanisloki = 0 — never invent an absent commentary).
- Verse text + Leonov подстрочник: sibling `SamudraManthanam/web/corpus_builder/jsonl/05_ramayana-sundarakanda.jsonl`
  (`#ru` = Leonov's own literal layer).
- **Dedup baselines (all three, non-negotiable):** (1) the подстрочник; (2) **Leonov/Kostina's OWN
  1,058 notes** — `data/leonov_own_notes.json` (from `extract_leonov_notes.py`), model II tier-1 and
  the ~36% benchmark; (3) the Phase-1 apparatus (`data/sundara_commentary_to_add.json` 788 notes +
  `data/lexical/ch{N}.json`). The 2026-07-01 pilot lacked baseline (2) and 9/16 candidates overlapped
  Leonov's own notes — **the drafting prompt MUST include each sarga's Leonov notes.**
- Edition context: the apparatus annotates the **southern vulgate** (what Leonov translates), not the
  critical text. `scripts/compare_editions.py` → `data/edition_comparison/`; 196 southern-only contiguous
  passages are standing footnote candidates («в критическом отсутствует»). Sargas align **by content**
  (`per_sarga_aligned`), the by-number table is raw and misleading.

## Pipeline (STEP 1–4, per PHASE2_METHOD.md)

1. **Segment (deterministic):** `python scripts/extract_yellow_sargas.py [NN NN ...]` →
   `data/analysis/sundara_commentary_segmented.json` (per-verse bundles: verse + подстрочник + up to 3
   commentaries). Caveats carried into the gate: merged-range markers (`5.35.810` = vv. 8–10, flagged
   `ambiguous_marker`) and marker-offset glitches (the 5.36.45 case) — **verse_ids must be verified
   against print**; segmenter alignment ≈81%, improvement plan is PHASE2_METHOD.md §11 (not yet built).
2. **Draft (LLM, hybrid per [D-P2-4]):** many **small parallel write-only subagents (~2–3 sargas each,
   no git in-agent)**, each emitting `data/analysis/phase2_pilot/sarga_NN_candidates.json` with the §3.1
   style contract: 1–3 sentences, ЛП register, **contrastive-first** («в „Тилаке“ — X; в „Широмани“ — Y;
   перевод следует…») where ≥2 commentators diverge, commentator named in-note («по Тилаке», [D-P2-2])
   + machine `source_commentary` field, short IAST pratīka ≈12% ([D-P2-3]), Kazansky Б/В (realia → В,
   never Б), required `why_proposed` clause, every note `review_required:true`. Backend: driven inside
   Claude Code the drafting agents need no API key; the standalone `scripts/annotate_batch.py` path is
   **DeepSeek via the openai-compat backend** (`LLM_BASE_URL=https://api.deepseek.com`,
   `LLM_MODEL=deepseek-chat`, `LLM_API_KEY` injected at runtime — no dotenv) — **never ask for an
   Anthropic key**. Reject discipline §3.2: expect ~2/3 of commentary to merely restate the подстрочник;
   a HIGH accept rate is the warning sign (pilot accepted 6.3%).
3. **Judge (STEP 2b, optional but preferred at scale):** fresh-instance, pointwise, refute-framed
   rubric scoring (faithfulness gate = must-be-2; verdicts keep/edit/park/reject/flag_anchor) merged
   into the candidates JSON — the judge ranks, the human gates; nothing loses `review_required`.
4. **Merge + gate (deterministic + human):** `python scripts/merge_phase2_pilot.py` →
   `pilot_candidates.json` + `pilot_rejected.json` (every rejection keeps its reason bucket — rejected
   logs are evidence, never destroy). Then emit the **interactive HTML review gate** with the shared
   emitter — `render_review_sheet(items, config)` from
   [`csl-pyutil`](https://github.com/sanskrit-lexicon/csl-pyutil) v0.3.0/v0.3.1, per
    [/review-sheet](https://github.com/gasyoun/claude-config/blob/main/commands/review-sheet.md). **Hand-rolled sheet HTML is banned**, and markdown checkbox sheets
   stay banned; `PILOT_REVIEW.md` remains the read-only fallback.
   - **Item shape** `{id, filt, title, badges, question, panels, note_placeholder}` (+ `title_href` to the
     verse, V4): `question` = the verse + подстрочник + the candidate note with its commentators' readings
     (enough to judge without opening other files); `filt` = the source commentary / judge verdict;
     `panels` = the dedup evidence (Leonov-own-note overlap, Phase-1 hit) and `why_proposed`.
   - **Mandatory config (V1–V8):** `show_ids: True` (V3), `note_min_height_px: 88` (V6), `save_as` bound to
     the destination `_decisions.json` (V8), Russian card text wrapped with `csl_pyutil.mark_cyrillic()`
     after escaping (V7 — never highlight the Devanagari/IAST), `generated` passed explicitly. The emitter
     carries the v0.3.1 dark-theme contrast fix (`color-scheme: dark` + a forced-dark note textarea).
   - **Three verbs only — ✅ approve · ❌ reject · ⏸ defer.** The old ✏️ *edit* button is banned: a
     requested rewrite is a **reject (or defer) plus the free-text note**, which is what the legend footer
     must say.
   - **Filenames** per the naming convention — `commentarystrategies-sundara-apparatus_sarga-NN-MM_review.html`
     + `commentarystrategies-sundara-apparatus_sarga-NN-MM_decisions.json` under
     `data/analysis/phase2_pilot/`. A generic `review.html`/`decisions.json` is banned.

   After MG/Kostina return the voted `..._decisions.json`: `python scripts/apply_phase2_decisions.py`
   grafts the approved notes into `data/sundara_ch{NN}_commentary_to_add.json` with
   `gated_by`/`gated_date` stamps (a note-requested tweak is applied before grafting, not auto-accepted).
5. **Validate + refresh:** `python scripts/validate.py` and `python scripts/derive_urn.py --check`
   after any data change; rebuild `data/sundara_book_stats.json` + the enriched HTML when totals move.
   Report HTML must carry the design-system markers (`css/commentary.css` link + literal
   `<main class="container">`) or `validate.py` fails — **the emitter-generated review sheet is exempt by
   construction** (it is deliberately self-contained, inline CSS+JS, opened from `file://`): exclude the
   `*_review.html` sheets from that check rather than hand-editing emitter output.

## Episodic Pāṇini cross-ref

For the rare epic verse whose note turns on a Pāṇinian grammatical crux (a parse the commentators
dispute in sūtra terms), consult [`/panini-commentary-corpus`](https://github.com/gasyoun/claude-config/blob/main/commands/panini-commentary-corpus.md) for the
commentator frame. **Episodic only** — Sundara is a «Литературные памятники» edition, not a grammar
edition; Pāṇinian commentary is not a standing layer here.

## Hard rules & environment

- Never «М.: Наука 2022» for Leonov (nonexistent volume — `validate.py` enforces); «Парибка» is the only
  oblique form; no 5th analytical axis; commentators may be named in the apparatus but never in an
  article TITLE (hard-rule #4). Known pre-existing violations to clean, not imitate:
  `scripts/sundara_ch1_enrich.py` + `data/sundara_ch1_corpus_relevance.json`.
- **Multi-actor repo:** a separate-account automation merges PRs to `main` and switches branches
  mid-session — verify end-state on `origin/main` directly, prefer branch+PR over direct main, and use
  a fresh `git worktree` when the shared tree may be contended. The host cycles long agents: small
  resumable write-only agents + central deterministic merge run by the orchestrator/human is the
  proven shape.
- Attribution obligation: any deposit/redistribution carries the Gita Supersite permission string
  (CC BY 4.0, `data/valmiki_PERMISSION.md`).
- Model provenance: report tier + exact version for every drafting/judging step (whatever actually
  drafts — DeepSeek `deepseek-chat` for standalone-pipeline drafts; an in-session Anthropic agent by
  its own tier+version). QA gates are deterministic Python, not an LLM judge.

## Stop conditions

Stop and report when: the pilot register is not yet approved and the request implies scale-up; a
segmentation glitch makes verse anchoring untrustworthy for a sarga; a requested commentary has zero
coverage for the target sargas; or `validate.py`/URN checks fail for reasons outside the change.

## Reporting

Sargas processed · verse-bundles segmented · candidates drafted/rejected (+ reject-reason buckets) ·
judge verdict split (if run) · review page path · density delta vs the ~36% Leonov-parity target ·
validation commands + results · what still needs the human gate.

_Mirrors the Codex skill `commentary-apparatus-ingest` (kept in sync via [`/claude-codex-skill-porter`](https://github.com/gasyoun/claude-config/blob/main/commands/claude-codex-skill-porter.md))._

_Skill authored by Fable 5 (`claude-fable-5`), 09-07-2026, H444 (Fable 5) - CommentaryStrategies commentary-apparatus-ingest skill._

_Dr. Mārcis Gasūns_
