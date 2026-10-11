---
name: "ab-experiment"
description: 'One isolated-variable controlled A/B/crossover experiment: frozen stratified sample, alternated order, pre-declared verdict rule, frozen-manifest artifact. Sonnet 5 (claude-sonnet-5); blinded grading Fable 5 (claude-fable-5). Trigger: "A/B this".'

---

# ab-experiment

Use this skill when the user asks to run the migrated source command `ab-experiment`.

## Command Template


# /ab-experiment <pipeline> <variable-under-test>

_Created: 17-07-2026 · Last updated: 17-07-2026_

Distills the discipline from three paired A/B experiments run back-to-back on the
PWG→RU recovery pipeline — [H841](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H841-Sonnet_SanskritLexicography_pwg-ru-recovery-policy-ab_13.07.26.md)
(recovery policy, Sonnet 5), [H842](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H842-Fable_SanskritLexicography_pwg-ru-prompt-quality-ab_13.07.26.md)
(prompt quality, blinded grading, Fable 5), [H843](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H843-Sonnet_SanskritLexicography_pwg-ru-window-size-ab_13.07.26.md)
(window size, crossover) — census candidate #13 in [SKILL_CANDIDATES_CENSUS_ALL_EXECUTORS_12_17_07_2026.md](https://github.com/gasyoun/Uprava/blob/main/archive/SKILL_CANDIDATES_CENSUS_ALL_EXECUTORS_12_17_07_2026.md).
All three sessions independently reinvented the same shape; this skill is that shape.

Argument: the pipeline under test, plus the one variable/config that differs
between arms (e.g. "recovery policy", "prompt A vs B", "window size 10 vs 20").

## Dedupe vs `/eval-harness`

`/eval-harness` runs **N times** to measure ONE variant's own noise floor
(mean ± spread, reproducibility). `/ab-experiment` runs **two-or-more variants**
against each other, under the SAME controls, to pick a winner. They compose:
if the noise floor isn't known, run `/eval-harness` first — a delta smaller
than the observed run-to-run spread isn't a real result, it's noise wearing a
verdict's clothes.

## Prerequisite

An A/B experiment optimizes an already-safe baseline; it is not how you
diagnose an unsafe one. Start only after the baseline pipeline has cleared its
own readiness/acceptance gate (`/acceptance-gate` or equivalent). If the
pipeline handles credentials, use one owner-authenticated profile and never
handle or record the credentials themselves.

## Phase 1 — Isolate the variable

- Freeze a stratified sample **before** running anything — stratify by whatever
  drives difficulty (input size, complexity, placeholder count). Hash it.
- Define arms A/B (/C…) that differ in **exactly one** variable. Prompt, exact
  model version, inputs, schemas, budgets, and the audit/scoring code must be
  byte-identical across arms — if two things differ, it isn't an A/B, it's two
  unrelated changes wearing one experiment's clothes.

## Phase 2 — Controlled execution

- Alternate presentation order (A→B / B→A, or equal-sized crossover blocks) so
  order and fatigue effects cancel rather than confound the winner.
- Run without promoting either arm's output — an A/B experiment observes, it
  does not ship, until Phase 4 promotes the winner as a separate step.
- Log model tier + exact version, calls, retries, latency p50/p95, and any
  quota/auth incidents per call, the same event/observability interface the
  pipeline already uses (reuse it — do not build a parallel logger).
- For qualitative arms (prose quality, translation fidelity), **blind the arm
  labels before grading**; reveal the map only after grades are recorded
  (H842's pattern) — an unblinded label is a thumb on the scale.

## Phase 3 — Pre-declared verdict rule

Declare the primary metric and every guardrail threshold **before** running,
not after seeing results — picking the rule after the data is in isn't
experimentation, it's rationalization. The house shape:

- One primary metric (e.g. audit-clean rate, clean subcards/hour, blinded
  paired preference).
- A guardrail band the challenger must clear even if it wins on the primary
  metric (e.g. "B wins only if fidelity-reject stays under 5% AND clean rate
  is within 2 points of A; otherwise retain A; an exact tie retains the
  current default").
- A hard NO-GO list that overrides the primary metric entirely: silent data
  loss, duplicate output, authentication failure, or malformed output on
  either arm kills that arm regardless of how well it scores.

## Phase 4 — Deliver

- Commit the frozen sample manifest + hashes, the runner/config for both arms,
  the machine-readable per-case results table, and a short written verdict.
- Do **not** commit the arms' generated outputs themselves (translations,
  cards, whatever the pipeline produces) unless the winning arm is being
  formally promoted — that promotion is a separate, explicit step from the
  experiment commit.
- If a winner is declared, flip it to the pipeline's sole default in that
  separate step, not silently inside the experiment commit.
- CHANGELOG entry (`/cut-release` reflex); a surprising delta or gotcha goes to
  `/findings-append`.

## Hard rules

- Exactly one variable changes between arms, full stop.
- The verdict rule (primary metric, guardrail band, NO-GO list) is written
  down before the first arm runs.
- Never promote a losing, unverified, or ungraded arm's output.
- Silent loss trumps every other metric — it is always NO-GO.

_Dr. Mārcis Gasūns_
