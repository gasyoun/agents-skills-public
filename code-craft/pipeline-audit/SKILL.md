---
name: "pipeline-audit"
description: 'Audit a data/build pipeline''s real capability - reconstruct the true call graph from code, census silent-failure sites, commit a capability memo. Triggers: "why does this fail silently". Fable 5 (claude-fable-5) tier, Opus 4.8 (claude-opus-4-8) fallback.'

---

# pipeline-audit

Use this skill when the user asks to run the migrated source command `pipeline-audit`.

## Command Template


# /pipeline-audit <repo-or-pipeline> [--post-incident <what-fired>]

_Created: 11-07-2026 · Last updated: 26-08-2026_

Distills the audit shape run four-to-five times (H012 (Fable 5) — [PWG architecture audit](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H012-Fable_SanskritLexicography_pwg_architecture_audit_02.07.26.md), H148 (Fable 5) — [MAX workflow handoff gap audit](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H148-Fable_RussianTranslation_max_workflow_handoff_gap_audit_03.07.26.md), H335 (Fable 5) — [pipeline capability audit](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H335-Fable_RussianTranslation_pipeline-capability-audit_08.07.26.md) — audit memos; H234 (Fable 5) — [PWG-RU agent orchestration failures fix](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H234-Fable_RussianTranslation_pwg_ru_agent_orchestration_failures_fix_06.07.26.md), H394 (Fable 5) — [code hardening](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H394-Fable_RussianTranslation_code-hardening_07.07.26.md) — post-incident hardening twins): census candidate C4 in [FABLE_FOOTPRINT_SKILLS_CENSUS.md](https://github.com/gasyoun/Uprava/blob/main/archive/FABLE_FOOTPRINT_SKILLS_CENSUS.md), built via [H578 (Fable 5) — Uprava pipeline-audit skill build](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H578-Fable_Uprava_pipeline-audit-skill-build_11.07.26.md). `/validator-recalibrate` owns one detector's thresholds; THIS skill owns the whole pipeline's architecture and its silent-failure surface. **Audit-only:** the deliverable is a memo + specs, not patches — remakes are their own (structurally unique) sessions.

Argument: the repo or the specific pipeline (entry script / stage chain). `--post-incident` names the failure class that just fired and switches Phase 2's depth to that class.

## Phase 1 — The real call graph

Reconstruct what actually runs from the code: entry points → stage order → what each stage reads/writes (files, DBs, APIs) → what invokes what (cron, CI, a human command). Diagram as a text table/tree in the memo. Where docs/`.ai_state.md` claim a different shape than the code, record the divergence — doc-vs-code drift is a first-class finding. Read-only agents (`Explore`) for the sweep; verify load-bearing edges by reading the actual call sites.

## Phase 2 — Silent-failure census

Walk every stage for the org's known killer classes: swallowed exceptions / bare `except`, unchecked subprocess exit codes, missing `encoding='utf-8'` on Windows-run IO (the recurring BOM/cp1251 class), unbounded network waits with no timeout, partial-write states with no atomicity, success messages printed regardless of outcome, hardcoded paths/repo lists that silently skip new members. One row per site: location (file:line) → class → what gets lost when it fires → observed-or-hypothetical. In `--post-incident` mode, exhaust the fired class across the WHOLE pipeline before touching others — the twin incident is usually sitting one stage over.

## Phase 2b — Required checks for an execution/money-risk pipeline

When the pipeline spends money or drives an executor (LLM calls, paid API, a bounded run), these four checks are **mandatory**, not optional — a pipeline audit that omits them is incomplete:

- **Execution-route parity** — the route a manifest/config *declares* (e.g. `execution_route`) is the route actually taken at the call site. A declared headless route that a code path can bypass into a Workflow/manual lane is a first-class finding (the H1110 (Opus 4.8) execution-route-parity defect class).
- **Declared-versus-enforced budget checks** — for every budget/limit block that is serialized (agent caps, cost ceiling, call count, timeout), find the code that *reads and enforces* it. A budget that is declared but never consumed (the 7,200 s runtime default vs a never-read 180 s manifest ceiling — a 40× breach) is exactly the silent gap this catches.
- **Data-at-rest invariants** — required fields present, no unresolved/placeholder tokens, no null-owner/synthetic rows, hash lineage and exact key delta intact, store backups preserved — verified on the store both before and after any promotion.
- **Census of side effects and cost gates** — enumerate every side effect (store write, TM rebuild, ledger append, backup) and every cost/quota gate, and confirm each is bounded and reversible or backed up. A billed side effect with no gate is a money-risk finding.

## Phase 3 — Capability memo + gap specs

Commit `docs/PIPELINE_AUDIT_<name>_<DD-MM-YYYY>.md` in the pipeline's repo (dated header, byline, full blob URLs): the call graph, what the pipeline CAN do today (capability inventory, honest), the silent-failure table, and a **ranked gap-spec list** — each spec concrete enough to hand to a build session (what to change, where, how to verify). Deliver via PR, auto-merge per the large-work rule.

## Phase 4 — Route, don't fix

Canonical-store corruption, null/placeholder backfills, or evidence-preserving
dataset restoration routes to `/canonical-data-repair`. This audit may locate
the source of corruption and map affected readers/writers, but it must not edit
canonical rows. Readiness remains `/acceptance-gate`; colliding repair lanes
remain `/dual-run-salvage`.

Each gap spec → a GTD row or a minted handoff (batch-mint when several); a measured non-obvious gotcha → `/findings-append` same pass. The audit session patches nothing — the one exception: a `--post-incident` run may land the single guard that closes the fired class, as its own separately-reviewed commit.

## Hard rules

- Code is the source of truth; every claimed edge is verified at its call site, every failure-site row carries file:line.
- Audit-only discipline — findings are routed, not patched (except the named post-incident guard).
- The memo states what was NOT audited (stages skipped, repos not local) — silent partial coverage is the same defect the audit hunts.

_Dr. Mārcis Gasūns_
