---
name: "red-gate-build"
description: 'Build ONE mechanical enforcement gate: fail-closed check + selftest with positive AND negative (the gate must actually refuse on bad input) + wiring into the enforcement surface (pre-commit / pre-push / close-path / cron / CI) + provenance marker + proof the refusal fires. Triggers: "put a gate on X", "make this fail-closed", "enforce Y mechanically". Mechanical build — OxAlpha; enforcement-surface choice is judgment — Sonnet 5 (claude-sonnet-5).'

---

# red-gate-build

Use this skill when the user asks to run the migrated source command `red-gate-build`.

## Command Template


# /red-gate-build <what-to-refuse>

_Created: 07-09-2026 · Last updated: 07-09-2026_

Distills the nine-session gate-building streak of 04–07-09-2026: the pre-commit config-escape schema gate ([H4080](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H4080-OxAlpha_claude-config_config-escape-schema-validation-precommit_04.09.26.md)), the drain-repo allowlist guard ([H4086](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H4086-OxAlpha_Uprava_drain-repo-allowlist-guard_04.09.26.md)), the drain-pool budget/lock-steal preflight ([H4095](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H4095-OxAlpha_Uprava_drain-pool-budget-lock-steal-preflight_04.09.26.md)), the threshold auto-gate ([H4111](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H4111-OxAlpha_Uprava_drain-threshold-auto-gate_05.09.26.md)), the sheet-evidence red gate — now standing adjudication policy ([H4114](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H4114-OxAlpha_Uprava_sheet-evidence-gate_05.09.26.md)), the four-leg preflight chain ([H4143](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H4143-OxAlpha_Uprava_drain-vote-chain_05.09.26.md)), the close-path effort gate ([H4146](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H4146-OxAlpha_Uprava_effort-tag-enforce-close_05.09.26.md)), the pre-push main-line plausibility gate ([H4157](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H4157-OxAlpha_Uprava_push-main-line-plausibility-gate_05.09.26.md)), and the 4-gate root-cause repair to 6/6 ([H4147](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H4147-OxAlpha_Uprava_failsweep-gate-repairs_05.09.26.md)). Approved **C47** in the skill-mine delta census ([SKILL_CANDIDATES_CENSUS_H4060_H4304_07_09_2026.md](https://github.com/gasyoun/Uprava/blob/main/SKILL_CANDIDATES_CENSUS_H4060_H4304_07_09_2026.md)), MG chat vote 5/5 approve 07-09-2026.

The discipline this owns: **a gate that has never refused is a suggestion.** One run of this skill ships a gate whose refusal is proven, not assumed.

## Phase 0 — What is refused, and who refuses it

- One sentence: the forbidden condition. Name the **enforcement surface** that owns the refusal (pre-commit hook, pre-push guard, close-path check, cron preflight, CI job) — a gate wired into a surface nobody runs is decoration.
- Fail-closed by default: missing data/permission/config = REFUSE, not pass. The escape hatch, if any, is a deliberate named env (`ALLOW_*_…=1`) that logs the trip.

## Phase 1 — Implement the check

- Deterministic, offline, fast (a pre-commit gate over seconds, not minutes). Read-only on the world except its own ledger/marker.
- Root-cause surface, not symptom: H4147 repaired four failing gates at source — never silence, never bless-as-advisory (MG red-invariant).

## Phase 2 — Selftest: positive AND negative

- Positive control: a known-good input passes. Negative control: a known-bad input is refused — **the refusal must actually fire** (assert exit code / blocked action, not just the message).
- A gate with no negative control is unverified by definition; this is the single most-skipped step in the nine prior sessions.

## Phase 3 — Wire + provenance + ledger

- Wire into the enforcement surface (hook config, close-path call site, CI yaml) and record the wiring in the check's own doc.
- Blockers record their trips: a wired PreToolUse/PostToolUse blocker calls `hook_block_ledger.record_block()` at its exit-2 decision site (estate law since H3653) — "liveness marker ran" is not "blocked".
- Register in the repo's gate inventory (failsweep/`docs/FAILSWEEP_LATEST.md` for Uprava) so the next census can see it.

## Phase 4 — Prove it fires in anger

- Force one real trip post-wiring (bad input through the real path) and keep the receipt. H4147's corollary: when a gate flips red mid-run from a sibling's churn, diagnose or re-run — never green it by hand.

## Dedupe fence

- [`/gate-judge`](https://github.com/gasyoun/claude-config/blob/main/commands/gate-judge.md) — judges an ALREADY-LOCKED gate's outputs (PROCEED/HOLD); never builds.
- [`/acceptance-gate`](https://github.com/gasyoun/claude-config/blob/main/commands/acceptance-gate.md) — multi-axis readiness evidence packet for a pipeline.
- [`/computed-table`](https://github.com/gasyoun/claude-config/blob/main/commands/computed-table.md) — the derive-don't-store `--check` parity gate, one narrow case.
- [`/hygiene-track`](https://github.com/gasyoun/claude-config/blob/main/commands/hygiene-track.md) — census tracks (lint/secrets/md) that may END in gates, not build one.
