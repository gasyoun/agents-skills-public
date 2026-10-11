---
name: "eval-harness"
description: 'Eval/reproducibility harness around a gold benchmark: N-run averaging with variance, temp-0 probe. Opus 4.8 (claude-opus-4-8)/Sonnet 5 (claude-sonnet-5); blinded grading Fable 5 (claude-fable-5). Triggers: "eval harness", "reproducible benchmark".'

---

# eval-harness

Use this skill when the user asks to run the migrated source command `eval-harness`.

## Command Template


# /eval-harness <pipeline-or-gold-set>

_Created: 11-07-2026 · Last updated: 18-09-2026_

Distills the harness discipline from H072 (Fable 5 — RuWritingStyles N=5 harness: flash detection 0.96 / pass-rate 0.48 — a gap ONE run could never see; temp-0 probe showed detection reproducible, revision NOT) and H152 (Fable 5 — csl-atlas R2 alignment harness): census candidate C8 in [FABLE_FOOTPRINT_SKILLS_CENSUS.md](https://github.com/gasyoun/Uprava/blob/main/archive/FABLE_FOOTPRINT_SKILLS_CENSUS.md). The gold set itself is `/gold-adjudicate` territory; THIS skill owns the machinery that runs it N times and reports honestly.

Argument: the pipeline/detector plus its gold benchmark (or the repo, if it has one gold set).

## Phase 1 — Pin what varies

Inventory every nondeterminism source before measuring: model **alias→version resolution** (pin the exact version — the H072 `deepseek-reasoner`→flash alias drift is the canonical trap), temperature, prompt template version, gold-set revision (pin by commit), and any task-routing config the runner actually consumes (H073 (Fable 5) gotcha: a routing file existed that the runner never read — verify consumption, don't trust presence).

## Phase 2 — The N-run core

- **N runs per case** (default N=5; state why if different), reporting per-case pass-rate AND aggregate mean ± spread — a single-run number on a nondeterministic pipeline is noise wearing a metric's clothes.
- **Temp-0 probe** as a separate mode: distinguishes "model can't do it" from "model does it unstably" — the two need different fixes.
- **Per-request wall-clock deadline** (the H072 v4-pro hang lesson): a hung request fails the case with a timeout verdict; it never stalls the sweep.
- Harness lives as tested package/CLI code in the repo (not a notebook), with resume-after-crash on long sweeps and every run logging model tier + exact version.

## Phase 3 — Baseline + regression wiring

Commit the first full N-run as the **baseline artifact** (per-case table + aggregates, dated, provenance) — future runs diff against it. Wire the cheap deterministic slice into CI/the repo's test suite where feasible; the expensive LLM sweep stays on-demand with its invocation documented. Metric moves ≥ the observed run-to-run spread are real; smaller moves are noise — say so in the report.

## Phase 3.5 — DeepSeek harness quirks (U4, applied 17-07-2026 per [FABLE_FOOTPRINT_SKILLS_CENSUS.md](https://github.com/gasyoun/Uprava/blob/main/archive/FABLE_FOOTPRINT_SKILLS_CENSUS.md) refresh3)

Measured on the H730 (Fable 5) defgen run ([Uprava FINDINGS §68](https://github.com/gasyoun/Uprava/blob/main/FINDINGS.md), 2,000+ calls) — pin these before running any DeepSeek arm, since both cost retries if hit blind:

- **`deepseek-reasoner` rejects `response_format` outright.** The org-canonical `build_corpus_lexicon.deepseek()` pins `response_format={'type':'json_object'}` + `model='deepseek-chat'`; sending that body to `deepseek-reasoner` 400s. A reasoner arm needs the field dropped and JSON parsed out of free text (`re.search(r'\{.*\}', raw, re.S)`).
- **`deepseek-chat` at temperature 0 in JSON mode can ramble past `max_tokens` on high-polysemy Sanskrit headwords** (`soma`, `giri`, `Seza`), returning truncated, never-closing JSON — deterministically, on every retry, so a resumable-skip loop never heals it. `max_tokens=8000` recovered most cases; for the remainder, cap prompt length and salvage the `"gloss": "…` string prefix out of the truncated JSON, flagging the row as salvaged rather than clean.
- Residue: strip-and-retry null rows once, then treat repeat offenders as **deterministic truncation, not transient failure** — retrying identically will not heal it.

## Phase 4 — Deliver

PR with harness + baseline (auto-merge per the large-work rule); CHANGELOG entry → `/cut-release` reflex. Measured gotchas (alias drift, hang classes, variance surprises) → `/findings-append`. If the repo has a goal gated on the metric, re-run its signal and update the goal's evidence.

## Hard rules

- Never report a single-run number as the metric; always N + spread, with N and pins stated in the artifact.
- Every alias resolved to an exact version in logs and reports; unconsumed config flagged, not assumed.
- The harness measures — it never silently patches the pipeline to make numbers pass; fixes are separate PRs diffed against the baseline.

## Mode — rigor adjuncts (U65, skill-mine H4542–H5153 vote 18-09-2026)

Before a number is trusted, the instrument is probed; before a metric moves, its significance is computed. Invoked as `--mode rigor` with a target lane.

**Judge-bias probes** (the judge is the instrument):
1. **Length probe** — correlate judge scores against answer length over the *existing* artifacts (no new spend); a high score tracking long answers is a rubric defect to report, not a model property.
2. **Position probe** — one bounded re-judge pass with the candidate order swapped; report the verdict-flip rate. **The control is mandatory and paired**: a fixed judge on the same items, pre-registered design (hypothesis + swap protocol committed BEFORE the run) whenever the probe is correcting a prior confounded inference (the H5010 correction of H4658 is the precedent — an unpaired swap cannot separate position bias from judge noise).
3. Disagreements route to human adjudication, never silently averaged.

**Metric adjuncts** (the metric is the instrument):
4. **Paired significance beside an existing one-sided metric** — exact McNemar + paired bootstrap CI on the SAME items; state what is noise (H4656's own selftest: 70 vs 73 % at n=100 is noise; a 19/20 paired flip is significant).
5. **Reliability beside pass@k** — pass^k (all k of k succeed) recomputed from existing N-run artifacts, zero new API spend; report both, never replace pass@k.
6. Every adjunct lands as tested code in the lane's tooling (e.g. `paired.py` beside `wilson.py`), with `--selftest`, and flips the owning index row.

Evidence: H4658 (length + position probes) · H5010 (pre-registered same-judge paired correction) · H4656 (`paired.py`) · H4657 (pass^k). The N-run core, temp-0 probe and baseline wiring above are unchanged.

_Dr. Mārcis Gasūns_
