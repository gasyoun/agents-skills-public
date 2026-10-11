---
name: "precision-panel"
description: 'Upgrade a single-judge/prototype number to a panel-grade number: stratified sample of a derived tier, k independent judges + adversarial verify, per-judge label files committed beside the aggregate report, precision + CI per tier, then sweep every doc quoting the stale figure. Trigger: "precision panel", "panel-grade number", "3-judge this". Judges on Opus 4.8 (claude-opus-4-8)/Fable 5 (claude-fable-5); sampling + aggregation is committed-tool work.'

---

# precision-panel

Use this skill when the user asks to run the migrated source command `precision-panel`.

## Command Template


# /precision-panel <target-tier-or-metric>

_Created: 04-09-2026 · Last updated: 05-09-2026_

Distills the nine-session family that kept re-deriving the same ladder: the acc/ncc adjudication wave ([H249](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H249-Sonnet_SanskritLexicography_acc_ncc_p1_fuzzy_crosswalk_06.07.26.md) + [H264](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H264-Sonnet_SanskritLexicography_acc_ncc_p2_adjudication_06.07.26.md), 06-07), the 49k agent adjudication ([H1657](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1657-Opus_SanskritLexicography_acc-ncc-p2-agent-adjudication-49k_26.07.26.md), [H1671](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1671-Opus_SanskritLexicography_acc-ncc-p0p1-ncc-key-repair-rerun_26.07.26.md), [H1951](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1951-Grok_SanskritLexicography_acc-ncc-p2-larger-sample_30.07.26.md)), the classifier precision pass ([H3394](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H3394-OxAlpha_Systema-Sanscriticum_classifier-precision-pass_23.08.26.md)), and the saru marker-head panel ([H3876](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H3876-Opus_SanskritRussian_marker-residual-morphology-recovery_02.09.26.md) → [H4007](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H4007-Opus_kosha_sense-recon-w2-precision-writeup_03.09.26.md) → [H4009](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H4009-Opus_SanskritLexicography_saru-marker-head-precision-panel_03.09.26.md) — «3-judge + adversarial verify, as gold/saru_gloss_precision_report.md did for dcs/vidyut/marker»). Approved **C45** in the full-estate skill-mine census ([SKILL_CANDIDATES_CENSUS_FULL_ESTATE_H1_H4059_04_09_2026.md](https://github.com/gasyoun/Uprava/blob/main/SKILL_CANDIDATES_CENSUS_FULL_ESTATE_H1_H4059_04_09_2026.md)), MG chat vote 9/9 approve 04-09-2026.

The discipline this owns: **a prototype or single-judge number is not a publishable number.** One run of this skill converts it — with committed evidence — into a panel-grade figure (n, precision, 95 % CI per axis) and retires every doc that quoted the old one.

## Phase 0 — What number is being upgraded, and who consumes it

- Name the stale figure: which report/doc/README quotes it, produced by how many judges (often one — a pilot pass, a classifier self-check, a single sampled batch).
- Name the derived tier or cell the new panel measures (`marker-head`, `hapax`, one classifier's accept-band…). Know the population per cell **before** sampling — small cells cannot fill a stratified quota; choose `k`/cell with that in the open.
- Confirm the aggregation machinery exists or commit it first (house pattern: `*_sample.py` + `*_aggregate.py` with a TIERS tuple — sampling and aggregation are committed, deterministic, rerunnable code; never chat-ephemeral).

## Phase 1 — Stratified sample from the real table

- Sample from a **fresh build** of the derived table, not a cached export (the H4009 trap: pick `k` knowing `mid`/`high` cells cannot fill).
- Stratify by the table's own frequency/tier axes; fix the seed and the per-cell quota in the command line or script args, so the sample is reproducible from the commit alone.
- Freeze the sample to a committed manifest (ids + cell + strat) before any judge sees an item.

## Phase 2 — k independent judges + adversarial verify

- **k ≥ 3 independent judges** — distinct model instances, no shared context, judging the same frozen items against the same rubric (name the axes: lemmatization / gloss / classification / …).
- One judge per item set runs an **adversarial-verify pass**: try to refute the majority, not to agree with it.
- Watch the axis that actually hurts. The H4009 lesson: a compound's Russian gloss attaches to its **head lemma** (a bahuvrihi inverts its head's sense) — the gloss axis failed while lemmatization passed. Do not report the healthy axis as the headline.
- Keep the standing convention intact: `partial` counts against precision (a denominator that silently excludes partials is a defect — name the denominator in the report).

## Phase 3 — Aggregate, commit labels, state the CI

- Aggregate with the committed tool: per tier × axis → n, precision, 95 % CI (bootstrap or Wilson — state which).
- Commit **per-judge label files** (`*_labels_<judge>.jsonl`) beside the aggregate report. A panel number whose per-judge labels are not committed is not panel-grade — it is unauditable prose.
- Never mix judge generations inside one figure; if a judge had to be rerun, record it in the report header.

## Phase 4 — The stale-figure sweep (the half everyone forgets)

- Grep the estate for every doc quoting the superseded number (report §-refs, ARCHITECTURE docs, README.meta backlog items, cross-repo quoters) and update each to the panel figure, with the report as the single citation source.
- The sweep is part of done: a new panel row sitting next to five docs still quoting the single-judge figure is a half-shipped panel.

## Hard rules

- **Denominator honesty:** state n per tier per axis; partials count against precision unless the report explicitly redefines and justifies it.
- **Unknown stays unknown:** a cell too small to fill its quota reports its real n, never a borrowed number from another cell.
- **Cost discipline:** k judges × n items is a paid multi-model run — state the spend shape before launching; reuse the existing sample/pipeline before minting new machinery.
- **Committed tools, not chat memory:** sampling, aggregation, and label files live in the repo next to the report; the skill orchestrates, it does not re-derive.

## Not this skill

- Adjudicating **two existing passes** into a gold set → [`/gold-adjudicate`](https://github.com/gasyoun/claude-config/blob/main/commands/gold-adjudicate.md).
- N-run averaging of a benchmark with variance/temp-0 probes → [`/eval-harness`](https://github.com/gasyoun/claude-config/blob/main/commands/eval-harness.md).
- The window-scoped judge sample inside a PWG close → [`/pwg-review-packet`](https://github.com/gasyoun/claude-config/blob/main/commands/pwg-review-packet.md).
- A bounded hand sanity-check of ~N rows → [`/spot-check-sample`](https://github.com/gasyoun/claude-config/blob/main/commands/spot-check-sample.md).

## Delivery

- Merged PR: aggregate report gains the new tier row (n, precision + CI per axis); per-judge label files committed; sampling/aggregation script changes included.
- The stale-figure sweep list (every doc updated, each linked) in the close-out.
- Report: PR link, per-tier table, judge generations used, spend shape, model tiers + exact versions.

_Dr. Mārcis Gasūns_
