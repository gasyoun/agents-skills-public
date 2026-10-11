---
name: "reader-layer"
description: 'Build ONE graded-reader layer the kosha way: feed-vs-decoy verify, vendored decoders, validated alignments, honest residue, manifest row. Sonnet 5 (claude-sonnet-5) layers; QA/rights Opus 4.8 (claude-opus-4-8). Trigger: "build the sandhi layer for X".'

---

# reader-layer

Use this skill when the user asks to run the migrated source command `reader-layer`.

## Command Template


# /reader-layer

_Created: 17-07-2026 · Last updated: 26-08-2026_

The kosha graded-reader discipline: one layer goes in, a registered dataset + a reader affordance comes out. Generalized from the Gītā-gold arc (W0 master → [H871](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H871-Sonnet_kosha_gita_reader_all_18_adhyayas_13.07.26.md) (Sonnet 5) reader → [H872](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H872-Sonnet_kosha_gita_sandhi_layer_13.07.26.md) (Sonnet 5) sandhi → [H873](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H873-Sonnet_kosha_gita_morphology_compound_gold_13.07.26.md) (Sonnet 5) morph gold → [H874](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H874-Opus_kosha_gita_inflection_engine_qa_13.07.26.md) (Opus 4.8) engine QA → [H875](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H875-Sonnet_kosha_gita_russian_etymology_layer_13.07.26.md) (Sonnet 5) RU/etymology → [H876](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H876-Sonnet_kosha_gita_root_preverb_semantics_13.07.26.md) (Sonnet 5) upasarga semantics) plus the reading-pack and crossref siblings ([H848](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H848-Sonnet_kosha_dcs_reading_packs_nala_gita_13.07.26.md) (Sonnet 5), [H896](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H896-Sonnet_kosha_bloomfield-rv-crossref_13.07.26.md) (Sonnet 5)).

**Design call (the census's own open reservation, ruled 17-07-2026 by Opus 4.8 `claude-opus-4-8` under [H1128](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1128-Opus_claude-config_skill-build-reader-layer_17.07.26.md)):** the 12 evidence sessions' layers DO vary structurally — but only in the extraction body. Everything around it is invariant and is where every one of those sessions spent its risk: feed verification, master-first sourcing, alignment validation, honest residue, manifest registration, reader surfacing. So this is **one skill with a family-template Phase 3** (§ Layer types below), not a skill per layer and not one skill per corpus. A new layer type earns a bullet in Phase 3, never a new skill.

Argument: the layer + its corpus/work (e.g. `sandhi gita`, `reading-pack MBh 3.50`, `crossref RV`). Resolve the current W-stage from the roadmap + the manifest before assuming a fresh start.

Evidence base: H871 H872 H873 H874 H875 H876 H836 H837 H838 H839 H848 H896 (census row #21 — 12 sessions, kosha; Sonnet 5 ×11 · Opus 4.8 ×1; the four infrastructure sessions: [H836](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H836-Sonnet_kosha_concordance_q2_parallel_passage_13.07.26.md) (Sonnet 5 — concordance Q2 parallel passage), [H837](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H837-Sonnet_kosha_heritage_anchor_reingest_13.07.26.md) (Sonnet 5 — Heritage anchor reingest), [H838](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H838-Sonnet_kosha_search_prefix_range_seek_13.07.26.md) (Sonnet 5 — search prefix range seek), [H839](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H839-Sonnet_kosha_pwg_scan_volume_disambig_13.07.26.md) (Sonnet 5 — PWG scan volume disambig)). Dedupe: `/crosswalk-build` owns two-dataset key reconciliation (a crossref layer CALLS it, never re-derives it); `/data-release` owns FAIR/DOI packaging of a finished layer; `/viz-page` owns feed→figure site pages; `/sandhi-annotate` and `/sandhi-gold-audit` own the sandhi rule table itself (this skill consumes their output as a layer); `/prior-art` is Phase 0's engine.

## Phase 0 — Prior art + feed verification (blocking)

- Run `/prior-art`. The layer's inputs are almost always already derived: the kosha manifest ([`data/manifest/datasets.json`](https://github.com/gasyoun/kosha/blob/main/data/manifest/datasets.json)), `corpus_lexicon`, `union_headwords`, the concordance join. Consume, don't rebuild.
- **Verify the feed against its decoys — the artifact, its encoding, and its coverage.** All three bit real sessions: `VisualDCS/src/dcs_full.sqlite` is a **0-byte LFS placeholder**, the real 920 MB DB is at `src/DCS-data-2026/dcs_full.sqlite` (H848); Bloomfield's `concordance.txt` is **UTF-16 despite the `.txt`** (H896); and the DCS Mahābhārata **silently omits the entire Gītā block** (`MBh 6, 23`–`6, 40`), so the Gītā half of H848 was undeliverable at any effort. Confirm size/encoding/row-count and confirm the TARGET's rows exist before planning the build.
- **Absent data parks, it does not block.** H848 shipped Nala 1 and surfaced Gītā-absent as an `@DECIDE` (external source needed) in the same pass. Never let the gated half hold the buildable half.

## Phase 1 — Master-first sourcing

- If the corpus has a master extraction (the Gītā W0 `data/gita/gita_gold_master.tsv`, 9,092 words × 21 fields), **every layer reads the master, not the source spreadsheet again**. Layers are siblings off one master; a second independent extraction of the same source is a defect.
- Fields the master DROPS are the layer's job to recover from the source, keyed back to the master (H875: the `Grammar` sheet's etymology column AG, aligned by verse + word-index). State which side you're on before extracting.
- Honour the corpus's locked decisions — read the roadmap's decisions section first (Gītā §7: garbled RU translit dropped, MIT, public, method-is-a-template). Do not revive what was ruled out.

## Phase 2 — Vendor the decoders, reuse the transcoders

- Numeric/coded master fields need their legend **vendored before decoding** (H873: the `Encode` + `Abbreviations` sheets, without which `code`/`tense`/`form_type` are meaningless integers). Validate the decoded output against the legend, not against your own reading of it.
- Transcode with the canonical helper — `sanskrit-util` per [SHARED_CODE.md](https://github.com/gasyoun/github-spine/blob/main/SHARED_CODE.md); DCS `token.lemma` is IAST, kosha keys are SLP1. Never hand-roll a transcoder, and use length-preserving `form_key()` normalization (never NFD+strip-Mn — it eats length + retroflex dots).
- Externally-sourced layers hit the rights gate FIRST: a permission record under `data/manifest/rights/` quoting the grant verbatim (marking any gap honestly rather than inventing connective text), plus the `LICENSE-DATA.md` entry, before the public-tier manifest row (H896/Franceschini).

## Phase 3 — Build the layer (family template — dispatch by type)

- **Reading pack** — sentences + ordered tokens for the chapter → resolve each token to its `/w/` card → pack data + passage viewer; unresolved tokens flagged, never dropped; loop the builder over the whole work rather than one chapter, and add navigation when you do (H871: 18 packs + chapter picker; H848: `MBh 3.50`).
- **Rule catalog** (sandhi) — per-word rule from the master + the frequency-ranked rule catalog from the source sheet → dataset + a standalone rules page + the reader hover affordance (H872).
- **Gold tagging** (morphology/compound) — decode every word to case·number·gender / tense·voice·person + compound type; the gold dataset is the input to the QA layer (H873).
- **Engine QA** (gold vs kosha's hybrid forms) — agreement % + a full divergence ledger of attested forms the paradigm disagrees with + candidate `disputed=1`/gap-fill corrections (H874). **Corrections route to a human `@DO`, never auto-applied.**
- **Sense/semantics panel** — root × preverb → sense (`pra-√vac` → declare), or gloss layers → a `/w/` card panel / a per-word toggle (H875 RU gloss, H876 upasarga).
- **Crossref** — join an external concordance/edition to the corpus keys via `/crosswalk-build`; also emit the external side as a standalone dataset in its own right (H896: 36,680 Bloomfield RV citations, reusable beyond the one join).
- **Drill surface** (L4 pedagogy HTML) — generate a **self-contained drill page** over a committed item bank (vocab frequency, Zaliznyak declension cells, semdom/thematic axes), not sandhi-only. Pattern: build script → `reading/.../drills/index.html` (or repo equivalent) + pytest/smoke that items load in-page + manifest/register. Evidence: [H1296](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1296-Opus_SanskritGrammar_sangram-attested-cell-declension-drills_19.07.26.md) (Opus 4.8 — attested-cell declension), [H1460](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1460-Sonnet_kosha_vocab-drills-inbrowser-page_22.07.26.md) (Sonnet 5 — vocab_drills.json page), [H1461](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1461-Sonnet_kosha_zaliznyak-declension-drill-surface_22.07.26.md) (Sonnet 5 — Zaliznyak declension drills), [H1462](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1462-Sonnet_kosha_semdom-thematic-vocab-axis_22.07.26.md) (Sonnet 5 — semdom thematic). **Sandhi item generation** stays `/sandhi-drill`; this bullet owns the **surface + multi-axis banks** that are not sandhi junctions.

## Phase 4 — Validate the alignment; never assume it positionally

This is the phase the arc's hardest lesson lives in — **the single rule to keep if you keep nothing else.** H896's naive `row1→pada a, row2→pada b` positional mapping looked obviously right and was **57% correct**; the file's printed half-line convention does not line up with grammatical pada lettering. The fix came from *inspecting real mismatches*, not from reasoning ahead: bucket a/b→half 1, c/d→half 2, then **independently validate every bucketed candidate** against the underlying text (strip parenthetical variant asides, `form_key()`-normalize, drop the pratīka's pausa-form final character since the continuous line carries external sandhi there, then check substring containment — a pada can sit mid-line). 57% → **85% validated**.

- Every join gets an independent validation against the source record. A positional assumption is a defect, however natural it reads.
- Build the validation from observed failures. Read the actual mismatches before theorising the rule.

## Phase 5 — Honest residue

- Report the rate — link rate per chapter (H871: ch1 99.6%), resolution rate, agreement %, validated rate — as a committed build report next to the dataset (`*_REPORT.md`), with the method and the residue named.
- **Stop chasing the tail when false-positive risk outweighs the gain**, and say so: H896's residual ~15% is genuine orthographic variance between two independent digitizations (anusvāra-vs-homorganic-nasal, consonant doubling) — spot-checked, documented, deliberately not chased. That sentence is the deliverable, not an apology.

## Phase 6 — Register + surface + sweep

- **Manifest row is mandatory** — `data/manifest/datasets.json` (tier, license, citation in the grantor's own requested wording where one exists), + `LICENSE-DATA.md`, + `PROJECT_INTERLINKS.md` if it feeds another repo.
- **A layer with no consumer is unfinished.** It lands on a reader surface in the same pass: the `/w/` card panel, the hover affordance, the passage viewer, the rules page. Build-verify the render.
- Sweep the records in the same pass: roadmap `@DECIDE` flipped with the ruling, `.ai_state.md`, `CHANGELOG.md` + `/cut-release`, `/findings-append` for any corpus gotcha the build surfaced, one PR per layer with auto-merge, model tier WITH exact version in every report.

## Gotchas

- The master is the source; layer datasets + packs are generated — never hand-edited.
- Decoys are the norm, not the exception: 0-byte LFS placeholders, `.txt` that is UTF-16, corpora that silently omit the exact target block. Confirm, don't infer.
- Unresolved tokens are flagged and counted, never silently dropped — the honest-residue policy is what makes the rate meaningful.
- QA findings are surfaced, not applied; a divergence ledger is a proposal to a human.
- kosha is public/MIT by default — but an externally-sourced layer is NOT public until its rights record exists; run `/publish-safety-check` if in any doubt.

_Dr. Mārcis Gasūns_
