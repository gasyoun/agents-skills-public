---
name: "shadow-bench"
description: 'Adopt-or-reject ONE external candidate (model, tool, MCP, service) via a shadow benchmark: freeze a sanitized corpus + state the incumbent floor → measure shadow-only (never prod) with pre-registered GO/NO-GO thresholds written BEFORE the run → selftest-first runners → dated JSON+MD report → wiring stays a separate unit even on GO. Triggers: "should we adopt X", "benchmark the new model", "shadow bench". Tier: OxAlpha.'

---

# shadow-bench

Use this skill when the user asks to run the migrated source command `shadow-bench`.

## Command Template


# /shadow-bench <candidate> [task]

_Created: 01-10-2026 · Last updated: 01-10-2026_

Distills the jev shadow-bench wave — 7 sessions (H5274–H5280: sufler shadow bench, client-wrapper selftest, claims pre-verifier with Brier + cost/1000, recall rerank vs hybrid baseline, samasa subtype choice, defgen gloss rerank, lead scoring shadow; ≥3 ✅ executed by OxAlpha), 23–24-09-2026. Approved **C55** in the skill-mine delta census ([SKILL_CANDIDATES_CENSUS_H5156_H5567_01_10_2026.md](https://github.com/gasyoun/Uprava/blob/main/SKILL_CANDIDATES_CENSUS_H5156_H5567_01_10_2026.md)), MG chat vote 10/10 approve 01-10-2026.

The discipline this owns: **an adoption verdict is a measurement protocol, not a vibe — the floor, the thresholds and the corpus are frozen before the first candidate call, and "floor not beaten" ships as a durable NO-GO, not as a shrug.**

## Phase 1 — Freeze the frame

- Frozen sanitized corpus (152-FZ fenced: no personal data ever leaves the machine; synthetic or public data only) + the incumbent's **explicit floor** measured on the SAME corpus (the jev floor was YAML+regex at 93% — a dumb-but-honest baseline beats no baseline).
- Decide the metric set NOW: quality (accuracy/F1), latency, and cost per unit (cost/1000 in jev). Write them down with the corpus.

## Phase 2 — Pre-register the verdict rule (before any candidate run)

- Thresholds are written and dated BEFORE the run: jev used recall ≥0.99 + Brier calibration + cost/1000 for the claims pre-verifier, and nDCG@5 ≥ +5% vs the hybrid baseline for the rerank arm.
- **floor-not-beaten = durable NO-GO note** — a candidate that cannot beat the floor ships a dated NO-GO that the next census/proposal must cite, not re-argue.

## Phase 3 — Shadow-only measurement, selftest-first

- Shadow means shadow: never point the candidate at prod traffic, prod DBs or live users. Wiring a GO into prod is a SEPARATE unit (its own handoff), even when the bench says GO.
- Runners are selftest-first: `--selftest` runs the full path offline and must PASS before any real call — a bench that cannot run offline will waste its window debugging transport.
- 152-FZ fences on every output: dated JSON + MD report committed next to the runner; the numbers must be third-party recomputable.

## Phase 4 — Report + verdict

- Dated report: corpus manifest + hashes, floor numbers, candidate numbers, the pre-registered rule, and the verdict **citing the rule** (GO / NO-GO / INCONCLUSIVE — inconclusive is never quietly rounded to GO).
- On GO: stop. Open the wiring unit as a separate handoff with the bench report attached as evidence.

## Dedupe fence

- [`/eval-harness`](https://github.com/gasyoun/claude-config/blob/main/commands/eval-harness.md) — builds an N-run harness around an existing pipeline/gold set; no adoption verdict, no floor.
- [`/ab-experiment`](https://github.com/gasyoun/claude-config/blob/main/commands/ab-experiment.md) — isolated-variable A/B on a LIVE pipeline; this bench never touches live.
- [`/sli-probe-build`](https://github.com/gasyoun/claude-config/blob/main/commands/sli-probe-build.md) — SLI/probe receipts on infrastructure, not candidate adoption.

_Dr. Mārcis Gasūns_
