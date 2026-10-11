---
name: "telemetry-first-parking"
description: 'Park a judgment call instead of guessing: write the decision down, build a small dated JSONL accrual + a nightly watcher, pre-register the verdict rubric (thresholds BEFORE the data), then rule mechanically when N days of rows exist. Trigger: "park this decision", "we will decide when the data arrives", "отложить решение до данных", "accrue telemetry first". Approved C51, skill-mine H4305-H4541, MG chat vote 11-09-2026.'
---

# telemetry-first-parking

Use this skill when the user asks to run the migrated source command `telemetry-first-parking`.

## Command Template


# /telemetry-first-parking <decision>

_Created: 11-09-2026 · Last updated: 03-10-2026_

Distills the 08-09-2026 cluster: [H4391](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H4391-OxAlpha_Uprava_script-telemetry-runtime-instrument_08.09.26.md) (script runtime telemetry to settle the parked tools/mint split) · [H4401](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H4401-OxAlpha_Uprava_telemetry-nightly-watch_08.09.26.md) (nightly watcher feeding the decision into the daily `--land` packet) · [H4403](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H4403-OxAlpha_Systema-Sanscriticum_storage-check-jsonl-series_08.09.26.md) (the 04:20 verdict persisted as a JSONL series so the H4291 calibration window accrues data) · [H4420](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H4420-OxAlpha_Uprava_telemetry-two-box_08.09.26.md) (two-box merge + preregistered verdict rubric v2 + EVALUABLE auto-GTD loop-closer). Approved **C51** in the H4305–H4541 skill-mine census ([SKILL_CANDIDATES_CENSUS_H4305_H4541_11_09_2026.md](https://github.com/gasyoun/Uprava/blob/main/SKILL_CANDIDATES_CENSUS_H4305_H4541_11_09_2026.md)), MG chat vote 11-09-2026.

The discipline this owns: **"we'll decide when we have data" is not a decision deferred, it is a decision to never decide** — unless the accrual, the watcher and the rubric are built in the same pass.

## Phase 0 — Name the decision and the datum

- Write the decision in one sentence, the two (or three) candidate outcomes, and the datum that discriminates them (runtime per script family, storage verdict per day, mint/close ratio…).
- Name the consumer: who will read the verdict, and where it lands (a GTD `@DECIDE` row is the default parking surface; a doc if it outlives the row).
- Mint the parked decision as a hypothesis row via [`/hypothesis-mint`](https://github.com/gasyoun/claude-config/blob/main/commands/hypothesis-mint.md) — QUESTIONS_LOG gets the claim → data → metric → baseline shape, so the eventual verdict lands in the win-rate ledger instead of evaporating in chat (wired H5790, 03-10-2026).
- If the datum already exists somewhere — stop, read it, decide now. This skill exists for data that does **not** exist yet.

## Phase 1 — The accrual (small, dated, append-only)

- One JSONL line per run: `{ts, subject, value(s), source}` — append-only, one file, in the repo that owns the decision (or `Uprava/data/` for cross-repo ones).
- The writer is a committed script with a selftest; **the series is regenerable, the file is the evidence** (H4403's whole point: a verdict that is only printed to a log accrues nothing).
- Keep it small: a handful of fields; the accrual's job is to be readable in N weeks, not to be a data warehouse.
- Two-box work merges by the same `{ts, subject}` key (H4420) — never two parallel series.

## Phase 2 — The watcher (so the accrual actually accrues)

- Schedule the writer where the estate's watchers already live (`launchd` on the Mac, Task Scheduler on Windows, `cron`/`systemd` on the hosts — per `/n8n` for .91 host-ops).
- Quiet hours and failure paths named: a watcher that dies silently is the H4391 defect class; fail-open logging + a dated GTD row if it goes silent >3 days.
- One watcher per decision — never a shared "telemetry daemon".

## Phase 3 — Pre-register the verdict rubric

- Thresholds, window length, and the tie/insufficient-data rule are written **before** the first row is read, into the same doc/handoff as the decision.
- The rubric names the automatic verdict shape: `EVALUABLE` (rule mechanically) vs `UNEVALUABLE` (extend window / new datum / escalate) — H4420's v2 rubric + auto-GTD loop-closer.
- No post-hoc threshold moves: if the rubric was wrong, write a v2 rubric with a new window, never retcon the old one.

## Phase 4 — Rule, record, close the loop

- When the window closes (or `EVALUABLE` fires), the ruling session reads the JSONL, applies the rubric, records the verdict with the row count and window, and closes the parking GTD row.
- When the verdict needs a corpus-side check, run it as a numbered probe via [`/hypothesis-probe`](https://github.com/gasyoun/claude-config/blob/main/commands/hypothesis-probe.md) — probe script + JSON sidecar, verdict to the register, outcome back to QUESTIONS_LOG (wired H5790, 03-10-2026).
- If the verdict is a human's (money/product), the mechanical half ends at a briefing with the numbers; the @DECIDE row stays human.
- Delete neither the series nor the rubric — they are the audit trail of why the decision is what it is.

## Hard rules

- **No accrual without a rubric** — data with no pre-registered threshold is a hobby, not a parking.
- **No rubric without an accrual** — a threshold with no rows never fires.
- **The datum is named before the instrument** — building a probe for an unnamed datum is the H4391 failure mode inverted.
- **Parking is not hiding:** every parked decision carries a dated GTD row and a window; an unowned park is a lost decision.

## Not this skill

- Production-acceptance packet with pre-registered gates (spends quota on a pipeline) → [`/acceptance-gate`](https://github.com/gasyoun/claude-config/blob/main/commands/acceptance-gate.md).
- Qualifying one model dispatch (served-vs-requested) → [`/model-router-canary`](https://github.com/gasyoun/claude-config/blob/main/commands/model-router-canary.md).
- Onboarding a newly released model to a lane → [`/model-lane-onboard`](https://github.com/gasyoun/claude-config/blob/main/commands/model-lane-onboard.md).
- One SLI with a firing receipt → [`/sli-probe-build`](https://github.com/gasyoun/claude-config/blob/main/commands/sli-probe-build.md).

_Dr. Mārcis Gasūns_
