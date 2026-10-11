---
name: "spot-check-sample"
description: 'Bounded, documented spot-check of a derived dataset/crosswalk against its primary source — pick sample size + method, hand-verify rows. Triggers: "sanity-check ~N rows by hand". Sonnet 5 (claude-sonnet-5)/Opus 4.8 (claude-opus-4-8)/Fable 5 (claude-fable-5).'

---

# spot-check-sample

Use this skill when the user asks to run the migrated source command `spot-check-sample`.

## Command Template


# /spot-check-sample

_Created: 04-07-2026 · Last updated: 03-09-2026_

Recurring pattern across independent handoffs — each one hand-rolling its own sample
size, selection method, and report format for the same underlying task: derive a
crosswalk / candidate-error list / mapping, then manually verify a bounded sample against
the primary source before trusting the full derivation. Evidence:
[H099](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H099-Sonnet_SanskritLexicography_heritage_inria_phase0_2_03.07.26.md) (Sonnet 5 (`claude-sonnet-5`) — Heritage/INRIA phase 0–2),
[H105](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H105-Opus_SanskritLexicography_heritage_phase4_morphology_forms_03.07.26.md) (Opus 4.8 (`claude-opus-4-8`) — Heritage phase 4 morphology forms),
[H106](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H106-Sonnet_SanskritLexicography_heritage_phase5_dico_gloss_layer_03.07.26.md) (Sonnet 5 — Heritage phase 5 DICO gloss layer),
[H111](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H111-Sonnet_kosha_ingest_heritage_forms_03.07.26.md) (Sonnet 5 — kosha ingest of Heritage forms),
[H128](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H128-Opus_csl-atlas_f4_dcs_shared_citation_errors_03.07.26.md) (Opus 4.8 — csl-atlas F4 DCS shared-citation errors)
each reinvent this from scratch with inconsistent rigor
(some say "≥20 rows", some "~10", selection method rarely stated; a sixth cited ID,
H143, was retracted same-day as a prior-art miss — the work was already shipped through
[sanskrit-lexicon/PWG#87](https://github.com/sanskrit-lexicon/PWG/issues/87). Its file is
real but keeps the pre-H###-dash naming,
[H143_pwg_pwk_indische_sprueche_crosswalk.md](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H143_pwg_pwk_indische_sprueche_crosswalk.md),
so the ordinary `H143-*.md` glob returns nothing and "retracted, documented" is
indistinguishable from "citation invented" unless you also read the ⚫ row in
[REGISTRY_ARCHIVE.md](https://github.com/gasyoun/Uprava/blob/main/handoffs/REGISTRY_ARCHIVE.md)).

**Not a replacement for:**
- [`/gold-adjudicate`](https://github.com/gasyoun/claude-config/blob/main/commands/gold-adjudicate.md)
  — that's full dual-pass reconciliation with κ, for when TWO independent annotation
  passes already exist.
- [`/review-sheet`](https://github.com/gasyoun/claude-config/blob/main/commands/review-sheet.md)
  — that's human approve/reject GATING on a batch of candidates, not source-verification
  of a sample.

This skill is for the narrower, more frequent case: ONE derived dataset, verified against
ONE primary source, on a bounded sample, to establish a confidence number before the
dataset is trusted or shipped.

Arguments: pointer to the derived dataset/crosswalk, pointer to the primary source it
should be checked against, optional sample size (default below).

## Phase 1 — Fix the sample size and selection method BEFORE looking at results

Pick a size proportional to what's being verified — the existing handoffs cluster
between 10 and 40 depending on how many disagreement/candidate rows exist and how
consequential errors would be (a citation-error candidate list warrants smaller/precise
sampling than a bulk crosswalk of thousands). Default to **20 rows** absent a stronger
signal. Selection method must be **stated and either random-seeded or exhaustive over a
natural stratum** (e.g. "all disagreement rows" or "random 20 of N with seed=<fixed>") —
never silently cherry-pick the easy cases. If the dataset has natural strata (e.g.
agreement vs disagreement rows, high-confidence vs low-confidence), sample within each
stratum separately and say so.

## Phase 2 — Verify each row against the primary source

For every sampled row: read the actual primary-source record (not a downstream summary
of it), and rule the derived row correct / incorrect / ambiguous. One line of reasoning
per row — this is what makes the confidence number defensible later, not just a count.
Do not batch-approve rows that "look plausible" without reading the source — that
defeats the purpose and has been the actual failure mode this pattern exists to catch.

**Fan the rows out to the read-only
[`sample-row-verifier`](https://github.com/gasyoun/claude-config/blob/main/agents/sample-row-verifier.md)
subagent — one row per spawn, in parallel.** Its contract is **five** lines and nothing
else — `VERDICT` + `ROW` + `SOURCE` + `REASON` + `SYSTEMATIC`. `ROW` is not optional trim:
it is the only field that maps a verdict back to the row it judged once parallel spawns
return out of order, so a fan-out that drops it cannot build the per-row table Phase 3
owes. The per-row
source-reading is exactly the large, dirty context this session must not inherit. If any
row comes back `SYSTEMATIC: yes`, stop the sweep and go to Phase 4's systematic-error
branch rather than finishing the sample. The verdicts are the agents'; the memo, the
accuracy estimate, the confidence bound and every write stay here.

## Phase 3 — Report

Emit a short memo (inline in the handoff/`.ai_state.md`, or its own file if the
verification is large): sample size, selection method, per-row rulings with reasoning,
resulting accuracy estimate (correct / sampled), and — critically — a stated
**confidence caveat**: what a spot-check at this N does and does not prove (a 20-row
clean spot-check over a 10,000-row dataset bounds error rate loosely, not tightly; say
the actual bound, e.g. via a simple binomial confidence interval, rather than implying
false certainty).

## Phase 4 — Decide and record

If the sample surfaces a systematic error pattern (not just isolated noise), STOP and
flag it rather than reporting a clean pass rate — a systematic error means the full
dataset needs a different fix, not a footnote. If the derivation is confirmed good
enough, mark it verified in the source handoff/GTD row with the memo linked. If not,
open the fix as its own next step rather than shipping the flawed derivation with a
caveat attached.

_Dr. Mārcis Gasūns_
