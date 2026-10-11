---
name: "validator-recalibrate"
description: 'Recalibrate or extend a corpus-signal detector (SanskritSpellCheck meter/n-gram shape) — measure flag rate, root-cause the dominant false-positive class before thresholds, never a tier promoter. Triggers: "recalibrate the validator", "flag rate too high", "extend the meter/ngram QA to <sections>", "39% is too many".'
---

# validator-recalibrate

Use this skill when the user asks to run the migrated source command `validator-recalibrate`.

## Command Template


# /validator-recalibrate <repo> <detector> [sections…]

_Created: 07-07-2026 · Last updated: 05-09-2026_

Distills the [H251](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H251-Sonnet_SanskritSpellCheck_chandas_validator_q1_2027_build_06.07.26.md) (Sonnet 5 — chandas validator build) → [H277](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H277-Opus_SanskritSpellCheck_chandas_validator_recalibration_07.07.26.md) (Opus 4.8 — recalibration) → [H289](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H289-Opus_SanskritSpellCheck_gretil_other_sections_pilot_07.07.26.md) (Opus 4.8 — GRETIL other-sections pilot) arc (SanskritSpellCheck meter validator: built at
39.1% non-clean → recalibrated to 15.9% → extended to new GRETIL genres). The shape
generalizes to any corpus-signal detector whose output feeds a human review queue:
the failure mode is always a flag list too large for any human ("no human will check
so many" — MG on 7,925 headwords vs the ~120 of a healthy queue), and the fix is
never "raise the threshold" first. Opus 4.8 (`claude-opus-4-8`) / Fable 5 (`claude-fable-5`) tier — every phase below has a
judgment fork.

## Phase 0 — scale sanity check (is recalibration even needed?)

Compare the detector's current flag count against what its human consumer actually
reviews. Order-of-magnitude rule from H277: a corroborating detector that flags
**thousands** of items is noise at scale; a healthy human queue is **dozens to low
hundreds**. If the count is already in range and precision spot-checks pass, stop —
report "calibrated, no action" and don't fish for changes.

## Phase 1 — root-cause from the tool's OWN diagnostics, never threshold-first

Before touching any threshold, break the non-clean set down by the underlying tool's
own diagnostic fields (H277's move: split `is_perfect=False` by empty vs non-empty
`problem_padas` — one grep-speed pass over the existing verdicts file, no corpus
re-run). You are looking for a **semantic misreading of a tool flag** ("is_perfect"
meant "most regular sub-pattern", not "valid"), which typically explains the bulk of
the rate in one stroke. Guards:

- **Reprocess existing indices; don't rebuild.** If only verdict/bridge logic
  changes, re-derive from the cached jsonl/index in seconds. Rebuild the corpus index
  only when per-item identification logic itself changed (H277: builders are
  resumable; raw corpora are gitignored + regenerable — never commit them).
- **Distinguish the two "empty diagnostic" cases** before folding either into clean:
  "named, recognized irregular variety" (valid — reclassify clean) vs "tool could not
  determine anything" (total failure — keep flagged). Verify on a hand-checked 20–30
  sample that the reclassification doesn't swallow genuine positives.

## Phase 2 — precision levers on what survives

Applied in H277 order, each optional on judgment:
1. **Localize the evidence bridge** — flag only the item(s) in the actually-defective
   span (the flagged pāda), not everything in the containing unit (the whole verse).
2. **Rarity-gate the output** — a common lemma near an anomaly is uninformative; only
   surface LOW-frequency items (DCS band ≤ 2 in the house pipeline). "Suspect + rare"
   is the informative combination, "suspect + common" is coincidence.
3. **Demote from vote to nudge** — read the aggregator's scoring closely: if the
   detector's hits increment a detector-count that promotes tiers, coincidental
   overlap silently inflates the top tier corpus-wide. Unless the post-fix hit list is
   demonstrably high-precision, move it to a score-only corroboration boolean that
   never counts toward promotion. **The H277 invariant: a corpus-signal detector is a
   ranking nudge, tier-C-by-default, never HIGH_PRECISION / never a tier promoter** —
   a similar signal was tried as a promoter before and rejected for surfacing
   coincidence as corroboration.

Re-run only the recalibrated detector + the aggregator (reuse the other detectors'
cached outputs); record flag counts + top-tier counts **before/after** in the repo's
detector README, and persist the comparison table durably (never chat-only).

## Phase 3 — extension to new sections/genres (the H289 half)

When the ask is "launch it on <new sections>", pilot before bulk:
1. **Fit triage first** (judgment): include sections the tool's model actually fits
   (Classical verse for skrutable/chanda); **exclude** known-mismatched material
   (Vedic meter/accent — the tool will mis-scan and inflate the rate); route
   prose-heavy sections to the running-text checker (ngram) instead of the meter
   validator. Only corpus-consuming detectors change per corpus — headword detectors
   and triage don't rerun.
2. **Sample large sections, never fetch-and-build whole epics** (~0.14 s/verse makes
   a full Purāṇa 30–50 min; MBh ~100k verses). Raw text + indices stay gitignored
   (third-party license, regenerable).
3. **Pilot acceptance = the invariant check**: does the non-clean rate on the new
   genre stay in the calibrated ballpark (~15–20% for meter), i.e. track
   ornateness/genre, not corruption? A spike = genre-fit problem → `@DECIDE` exclude
   vs per-genre tuning, parked to GTD — not a threshold hack. Eyeball 10–15 bridge
   hits for plausibility. Verdict → a short feasibility memo committed next to the
   detector README.
4. Scale-up after a passing pilot: per-section verdict/variety census, dataset via
   [`/data-release`](https://github.com/gasyoun/claude-config/blob/main/commands/data-release.md) discipline, feeds registered in `PROJECT_INTERLINKS.md` + the
   kosha manifest.

## Phase 4 — deliver

Error lists are **human-gated, never auto-applied**: dictionary-headword candidates →
[`/review-sheet`](https://github.com/gasyoun/claude-config/blob/main/commands/review-sheet.md); in-corpus (source e-text) typos → a separate upstream-facing list
(different consumer — GRETIL, not CORRECTIONS). PR the code/doc changes per the
repo's own delivery convention; update `.ai_state.md`; mirror any `@DECIDE` (genre
exclusions, promotion-policy changes) to GTD.

## When this isn't the right tool

- One-off precision estimate of an existing list → [`/spot-check-sample`](https://github.com/gasyoun/claude-config/blob/main/commands/spot-check-sample.md).
- Two annotation passes needing adjudication → [`/gold-adjudicate`](https://github.com/gasyoun/claude-config/blob/main/commands/gold-adjudicate.md).
- Building a brand-new detector from scratch → a minted handoff (this skill tunes and
  extends what exists).

_Dr. Mārcis Gasūns_
