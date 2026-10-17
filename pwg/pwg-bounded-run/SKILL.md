---
name: "pwg-bounded-run"
description: 'The PWG→RU bounded paid run — headless CLI, ONE profile, max-wide=1, --stop-before-promote so Opus 4.8 (claude-opus-4-8) reviews gate promotions. Needs GO from /pwg-live-gate. Triggers: "run the bounded window", after /pwg-live-gate. Do not use without a GO from /pwg-live-gate first, and do not use to close out the window afterward — that is /pwg-window-close.'
---

# pwg-bounded-run

Use this skill when the user asks to run the migrated source command `pwg-bounded-run`.

## Command Template


# /pwg-bounded-run

_Created: 18-07-2026 · Last updated: 22-09-2026_

Argument: a prepared lease id (or a path to an explicit nominal manifest-v2). Requires a **GO** row from **/pwg-live-gate** for the same profile slot — this skill does not gate the host; it spends the bounded window a passed gate authorized.

This skill is the **bounded execution** half: it runs a small, fully-bounded paid window through the **headless CLI (manifest v2)** — never a Workflow session — and stops **before** promotion for an Opus 4.8 (`claude-opus-4-8`) review. Working dir: `~/Documents/GitHub/SanskritLexicography/RussianTranslation`. Live executor tier = the exact model pinned in the manifest (resolve + record the version).

Orientation: [PIPELINE_HISTORY.md](https://github.com/gasyoun/SanskritLexicography/blob/master/RussianTranslation/PIPELINE_HISTORY.md) · the R9/R10 bounds live in [`bounded_staged_run.py`](https://github.com/gasyoun/SanskritLexicography/blob/master/RussianTranslation/src/pilot/bounded_staged_run.py) and [`headless_worker.py`](https://github.com/gasyoun/SanskritLexicography/blob/master/RussianTranslation/src/pilot/headless_worker.py).

---

## Preconditions (fail closed on any)

1. **/pwg-live-gate returned GO** for this exact profile slot, and the recorded manifest-v2 provenance (profile_slot · config_dir_fingerprint · execution_route · model version · validation method) matches the manifest you are about to run. A stale or mismatched gate row → STOP.
2. **Exact preflight** passes for the selected keys: `python src\pilot\perf_preflight.py <root/keys> --refuse-over-cost` — selected keys pass exact preflight and projected calls are within budget (G8). Over-ceiling keys are deferred, never squeezed in.
3. **One profile, max-wide=1** — the run binds a single logical profile slot and a single concurrent call.

## Bootstrap mode (the H1110 default — H1110 (Opus 4.8), PWG→RU post-H1080 audit fixes + skills + c4 restart) — hard bounds

Bootstrap forbids **retries, requeues, replacement candidates, and widening**. One prepared plan, one profile, one serial call lane, the worker's per-call kill ceiling (now **600 s** — the H2254 block below; it was 180 s at H1110, 300 s on 02-08→07-08), and fingerprint serialization (G9). No retry lane, no requeue `--no-tm` pass, no better-attempt swap, no `max-wide` above 1.

```powershell
cd ~/Documents/GitHub/SanskritLexicography/RussianTranslation
# --cwd is the CHILD's working directory: point it at an EMPTY scratch dir, never the repo.
$bare = Join-Path $env:TEMP ("pwg-bare-" + [guid]::NewGuid().ToString('N').Substring(0,8))
New-Item -ItemType Directory -Force $bare | Out-Null
python src\pilot\bounded_staged_run.py `
    --plan <prepared_plan.v1.json> --coord-dir <coord_dir> --coordinator src\pilot\coordinator.py --cwd $bare `
    --only-profile <slot> --max-accounts 1 `
    --max-windows 1 --max-calls <projected> --cost-ceiling <usd> `
    --canary-receipt <canary_receipt.json> --stop-before-promote --execute
```

> **H2158 (02-08-2026) — `--cwd` is a COST knob, and it used to say `.` here.** The CLI injects
> `CLAUDE.md` + git state into its cache prefix, and that injection is re-written at the premium
> create rate on **every** call. Measured on identical calls: repo cwd **$0.3036 / 26–29 s**,
> bare cwd **$0.2040 / 19–20 s** — **−33 % cost, −30 % wall clock**, no code change and no guard
> weakened. The manifest carries every input the worker needs, so nothing depends on running
> inside the repo. Do not "restore" `--cwd .`.

> H2157 (02-08-2026): `--execute` now REQUIRES both `--max-calls` and `--cost-ceiling` —
> a paid run refuses to start unbounded; `--allow-unbounded` is the explicit escape hatch.

> H2159 (02-08-2026): `--execute` now also REQUIRES `--canary-receipt` — the fresh GO
> receipt `canary_gate.py judge <canary_wf_output> --receipt <path>` writes after the
> /pwg-live-gate canary (verdict/age/profile validated mechanically, 6 h freshness).
> Precondition 1's "GO row" is no longer prose: the receipt IS the gate evidence.
> `--skip-canary-gate` is the explicit escape hatch.

> H1386 P3k: the previous form pasted `--max-agents 1 --timeout 180` here and died with
> "unrecognized arguments" — `--max-agents` exists only on `headless_worker.py`.
>
> **The `--timeout` half of that note was wrong, and H2254 (07-08-2026) corrects it.** It
> read "bounded's `--timeout` is the whole-**job** timeout (default 7200 s), not the per-call
> ceiling". It is neither: bounded passes the value **straight down** to
> `headless_worker.py --timeout` (via
> [max_account_orchestrator.py](https://github.com/gasyoun/SanskritLexicography/blob/master/RussianTranslation/src/pilot/max_account_orchestrator.py)),
> so it **is** the per-call ceiling, one level up. Two changes since:
>
> - the default is now the ceiling itself — **600 s** since 07-08-2026 (`DEFAULT_TIMEOUT_S = HARD_TIMEOUT_MS // 1000`, single-sourced from `execution_contract.PRODUCTION_HARD_TIMEOUT_MS`; it was 300 s on 02-08→07-08, 7200 before that) — not 7200;
> - a value **above the ceiling is REFUSED before any paid call** (`assert_timeout_within_ceiling`), not clamped down.
>
> Nothing bounds a whole job by wall clock. What bounds a job is `--max-calls` and
> `--cost-ceiling` (both required by H2157 above).

> **H2878 (issue #1680, FINDINGS §378) — the per-call ceiling is no longer the only kill.**
> A second, orthogonal bound exists in
> [`execution_contract.py`](https://github.com/gasyoun/SanskritLexicography/blob/master/RussianTranslation/src/pilot/execution_contract.py):
> `PRODUCTION_NO_OUTPUT_PROGRESS_MS = 90 000` bounds the longest stretch in which a spawn
> produced **no result bytes**, where `PRODUCTION_HARD_TIMEOUT_MS` bounds **total wall
> clock**. Different quantities, not a re-fit of the ceiling. A killed call now reports
> **which** of the two fired: `killed_reason` is `hard_timeout` or `no_output_progress`,
> never a bare `timeout`.
>
> **On this lane it is deliberately NOT armed, and that is the part to keep.** The window is
> derived from the spawn's own `--output-format` (`progress_window_ms_for`), and
> `headless_worker.WORKER_OUTPUT_FORMAT = 'json'` buffers the whole envelope to the end of
> the call, so the window resolves to `None` — **observe only, never kill**. Do not "fix"
> that by switching the lane to a streaming format: arming a 90 s stalled-output window
> against a buffered format kills healthy spawns wholesale (measured healthy range
> **49 404–511 908 ms, p50 189 327**). `None` is not "unmeasured" — every attempt still
> records `bytes_seen` and `quiet_ms`, which is what would turn a future arming decision
> into a reading instead of a bet. **Report both fields with the window's armed/observe-only
> state whenever a call is killed or comes back empty.**

> **Do not hand-set the supervisor's outer bound.** `max_account_orchestrator` tree-kills
> the worker at `headless_worker.wrapper_timeout_s(per_call) = per_call + 120 s`
> (`WRAPPER_TIMEOUT_HEADROOM_S`), and that headroom is load-bearing: it must sit **strictly
> above** the per-call ceiling. Set equal, the tree dies during teardown at the exact moment
> the last call reaches its own ceiling, `--status-out` is never written, and a legitimately
> killed call becomes a worker that vanished without a status file. The 120 s covers worker
> startup, manifest+preflight validation, prompt assembly and the atomic status/output
> writes — all outside the model call and therefore bounded by nothing else.

- `--only-profile <slot>` — enforces one manifest-bound logical profile slot; errors if `<slot>` is not a validated roster slot (never guess billing identity). The live slot is whatever /pwg-live-gate Step 0 rules — **c4 RETIRED, c1 sole live lane since 19-08-2026**.
- `--stop-before-promote` — writes the **durable, atomic, self-hashing `AWAITING_REVIEW`** terminal checkpoint (R10) **only after** a clean audit; it binds the execution-manifest + selected-keys hashes, lease/attempt, audit report, clean candidate, profile/route/model, and usage/cost+audit state. On restart it **prevents relaunch and promotion** — the checkpoint is the terminal state for an Opus 4.8 (`claude-opus-4-8`) to review, not a pause to resume blindly.
- An explicit nominal manifest-v2 (no lease) is driven the same way through `headless_worker.py` directly with `--only-profile <slot> --timeout 600` — **omit `--max-agents` on any multi-key manifest.** `--max-agents` is a **total spawn ceiling, not concurrency width**, so `--max-agents 1` starves a multi-key window: the worker refuses it outright (`LAUNCH_FUCKUPS C2_M50_W1_MAX_AGENTS1_2026-07-24`), and letting it through would re-create the only-b0 starvation class the [H1618](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1618-Codex_SanskritLexicography_unpaid-four-tracks-guards-parity_24.07.26.md) sync rule exists to prevent. Omitting it lets the manifest's own `max_translate_agents` / `max_heal_agents` budgets apply. **`--max-agents 1` belongs to the canary only** — one key that must finish in one spawn (`/pwg-live-gate` Step 2b) — and this line previously carried it by copy-paste from that step, which cost H3627 a launch. Also required on the paid v2 path: `--preflight <artifact>` whose `selected_keys` match `manifest.meta.selected_keys` exactly with `cost_gate.over_ceiling: false`, and `--execution-route=claude-cli-headless` at manifest-build time (`headless_cli` is refused). The execution-time fragment_tm gate refuses a null/ownerless owner **before any paid call** (R6). Same bare-cwd rule applies. **`--timeout 600`, never 180/300** ([H2173](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H2173-Opus_SanskritLexicography_pwg-audit-tail-g5-g8-g9-g10_02.08.26.md) G9, Opus 4.8 — PWG audit tail G5/G8/G9/G10): the effective per-call bound is `min(--timeout × 1000, budgets.timeout_ceil_ms, HARD_TIMEOUT_MS)` with `HARD_TIMEOUT_MS = execution_contract.PRODUCTION_HARD_TIMEOUT_MS` = **600 000** since 07-08-2026 (300 000 on 02-08, 180 000 before) — passing an old figure re-pins a retired ceiling by hand; read the constant from code. That is the defect behind a paid window that returned zero cards with 12 of 16 calls killed at exactly 180 s ([#983](https://github.com/gasyoun/SanskritLexicography/issues/983)).
- **Repair runs — a prepared requeue or a re-made promoted card ([H4527](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H4527-Opus_RussianTranslation_pwg-ru-cohort-live-acceptance-width2_10.09.26.md), 22-09-2026).** A frozen plan imports only `prepared` leases, so a `requeue_prepared` lease (from `coordinator.py prepare-requeue`) and a re-make of an already-promoted sub-card have no plan path. Name them instead: `bounded_staged_run.py --plan <plan.json> --repair-lease <id> [--repair-lease <id> …]` (repeatable; refused together with `--lease-id`). A `requeue_prepared` lease drains its prepared `<lease>::rqNN-<kind>` attempt through the unchanged `materialize_requeue`; a `prepared` lease of kind `defect-repair` (`coordinator.py claim --kind defect-repair --keys <sub-card> [--nominal]` then `prepare`) imports like any prepared lease — `--nominal` builds a no-PWG card the way `no_pwg_scale_plan` does, with `--no-tm` so the card being replaced cannot feed its own re-make. Any other state (plain prepared, promoted, unknown, duplicate) is refused, and an unnamed `requeue_prepared` lease stays invisible to a plan run. Every gate is unchanged: canary receipt, probe ration, `--max-calls` reservation, preflight validation. Run it dry first — `scope.repair_leases` lists each job id and its projected calls; the total must equal what the human ruled. Procedure: [RUN_FREQ_MAX.md](https://github.com/gasyoun/SanskritLexicography/blob/master/RussianTranslation/src/pilot/RUN_FREQ_MAX.md).
  A repair run is **not resumable once dispatched**: `--resume` refuses the now-`running`/`ready` lease (fails closed, no spend), so finish it by hand from `coordinator.py status` (a `ready` lease: `coordinator.py promote-ready --lease-id <id>`), and give each repair run a fresh `--checkpoint`. Any requeue of a `defect-repair` lease, transient included, stays TM-off. Merged via [SanskritLexicography#2310](https://github.com/gasyoun/SanskritLexicography/pull/2310), which superseded #2309.
- **Call shape stays ONE card per call; do not batch to save money.** Since H4054 (04-09-2026) this is the generator's structural default (`OUTPUT_BUDGET = 1` — a no-flag manifest already carries one key per `meta.batches` entry; `--output-budget=1` remains legal and explicit if you want to pin it by hand). Quota and the per-call wall (600 s since 07-08) bind in **opposite** directions — quota penalises *many* calls, wall clock penalises *large* ones — so whichever binds decides the shape, and as of 02-08-2026 it is wall clock. Batching additionally makes a single unevaluable call destroy per-card cost attribution for **all N** cards in its batch. A manifest that arrives batched anyway was prepared with an explicit experiment flag — treat it as a shape deviation, not the lane default. Ruling: [H2152 (Opus 4.8) — c4 quota call-shape audit](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H2152-Opus_RussianTranslation_c4-quota-call-shape-audit_02.08.26.md); structural default: H4054.
- **Record `cache_creation_input_tokens` and `cache_read_input_tokens` separately** for every call, plus the `cache_creation.ephemeral_1h…` / `…_5m…` bucket. Creation bills ~20× read and is the dominant charge; a combined total hides it, and the TTL bucket is what distinguishes "prefix unstable" from "cache expired" without an experiment. `subagent_tokens` is a **legacy misnomer** for the sum of the four fields — no subagents are involved. **Also record the two H2878 liveness readings** — `bytes_seen` and `quiet_ms` — on every attempt, and `killed_reason` (`hard_timeout` / `no_output_progress`) on every killed one; the cache fields price a call, these say whether it was alive.

## Terminal stop reasons (mechanical — do not override)

- **Success:** `STOP_CLEAN_TARGET` / `STOP_WINDOW_COUNT` / `STOP_CLEAN_QUOTA` / `STOP_CALL_COUNT` — the window met its bound cleanly and parked at `AWAITING_REVIEW`.
- **`STOP_COST_UNEVALUABLE`:** a clean result whose **actual-cost telemetry is missing** may be promoted, but the run then **MUST stop as `STOP_COST_UNEVALUABLE`** (G8) — `observed_cost_usd` stays authoritative; a missing-usage call flips `cost_evaluable=False` and the run does not continue on an unpriced window.
- **`STOP_PROFILE_UNBINDABLE`:** the profile could not be mechanically bound to the manifest's exact `profile_slot` + `config_dir_fingerprint` — **no paid call is made**; never ask a human to guess billing identity.

## Optional multi-account mode (documented, requires distinct fingerprints + global bounds)

A multi-account run requires **distinct `config_dir_fingerprint`s per account** and **global** call/account/cost bounds (`--max-accounts N` with per-account fingerprint validation); it is NOT the bootstrap default and is out of scope for the [H1110](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1110-Opus_SanskritLexicography_pwg-ru-post-h1080-audit-fix-skills-c4-restart_17.07.26.md) (Opus 4.8 — PWG-RU post-H1080 audit fix skills + c4 restart) c4-only bootstrap.

## Hand-off

- Terminal at `AWAITING_REVIEW` → hand the checkpoint to an Opus 4.8 (`claude-opus-4-8`) review (do **not** auto-promote). On approval, promotion + TM rebuild + ledger proceed via **/pwg-window-close**.
- Any `STOP_*` non-clean reason → report the reason, the manifest-v2 provenance, the measured usage/cost (or its absence), and stop. No retry, no requeue, no widening.

_Dr. Mārcis Gasūns_
