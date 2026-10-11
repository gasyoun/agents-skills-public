---
name: "cologne-integrity-audit-all"
description: Fan out cologne-data-integrity-auditor across Sanskrit Lexicon data/crosswalk repos to catch automation corrupting human-reviewed data (overlay wipes). Read-only. Sonnet 5 (claude-sonnet-5) / Opus 4.8 (claude-opus-4-8) / Fable 5 (claude-fable-5) tier. Do not use for security or injection vulnerabilities — that is /cologne-security-audit-all or /cologne-php-xss-sweep; this only fans out the data-integrity auditor for overlay-wipe or corpus-leakage risks.

---

# cologne-integrity-audit-all

Use this skill when the user asks to run the migrated source command `cologne-integrity-audit-all`.

## Command Template


# /cologne-integrity-audit-all

_Created: 03-07-2026 · Last updated: 05-09-2026_

Org-wide sweep for the bug class that has bitten twice already (csl-atlas R2 seeders, WhitneyRoots `apply_*`): an automated builder silently regenerating over human-reviewed canonical data. Uses the `cologne-data-integrity-auditor` agent, one per repo.

Entirely read-only. No PRs, no issues, no comments — findings go to a report + the hubs.

---

## Phase 1 — Target discovery

Candidate repos under `~/Documents/GitHub\`: every git repo that carries derived-plus-reviewed data — dict repos with `app_data`/overlay files, crosswalk repos (WhitneyRoots, csl-atlas, VisualDCS, IndologyScholars), correction repos (csl-corrections), and any repo whose `.ai_state.md` or README mentions "seeder", "checkpoint", "overlay", "promotion", or "human review". Skip pure-tooling and pure-docs repos. List the final targets before fanning out.

## Phase 2 — Load the known-bad baseline

Before spawning, collect the already-confirmed don't-rerun list so agents extend it rather than rediscover it:

- csl-atlas: R2 checkpoint/drift/promotion SEEDERS wipe human review (safe-to-rerun builders are documented separately).
- WhitneyRoots: `apply_*` and `corpus_verify` are don't-rerun.
- Whitney class I/VI, IV/passive: corpus signal must never be written into reviewed `app_data`.

Grep each target repo's docs/`.ai_state.md` for its own don't-rerun notes and pass them into the agent prompt.

## Phase 3 — Fan out

Spawn one `cologne-data-integrity-auditor` agent per target repo, ≤4 concurrent. Each prompt: the repo path, the known-bad baseline for that repo, and the ask — "find scripts/workflows that can overwrite or corrupt human-reviewed or canonical data; classify each as CONFIRMED-dangerous / suspicious / safe, with the exact file:line and the trigger scenario."

## Phase 4 — Verify before reporting

Route every CONFIRMED-dangerous finding through an `adversarial-verifier` agent before it enters the report — a false "this seeder wipes reviews" claim causes real workflow paralysis.

## Phase 5 — Deliverables

1. A report table: repo · script · verdict · trigger scenario · protection suggestion (guard flag, `--dry-run` default, checkpoint-before-write).
2. Append new confirmed findings to the overlay-wipe entries in `SanskritLexicography/FINDINGS.md` (Sanskrit-data repos) / `Uprava/FINDINGS.md` (infra), per `/findings-append` routing — update, don't duplicate.
3. Anything needing a human decision (e.g. "should this seeder be deleted?") → `@DECIDE` in `Uprava/GTD_NEXT_ACTIONS.md`.
4. Fixes are OUT of scope for this skill — propose them in the report; implement only on explicit follow-up.

Report model attribution per step (auditor tier+version, verifier tier+version).

_Dr. Mārcis Gasūns_
