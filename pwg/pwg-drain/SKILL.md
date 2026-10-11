---
name: "pwg-drain"
description: 'Run the PWG→RU per-root drain MECHANISM on the live lane — /pwg-live-gate gate → cost preflight → /pwg-bounded-run (headless CLI, manifest v2, --stop-before-promote) → /pwg-window-close. Triggers: "next PWG root", "resume the translation run". Do not use before confirming live-readiness — gate with /pwg-live-gate first; do not use to close a finished window — that is /pwg-window-close.'
---

# pwg-drain

Use this skill when the user asks to run the migrated source command `pwg-drain`.

## Command Template


# /pwg-drain

_Created: 04-07-2026 · Last updated: 05-09-2026_

Argument: a root name to drain, several roots, or `resume` / (empty) to take the current live-lane worklist head.

This skill is the **per-root/per-window drain mechanism** for the PWG→RU pipeline. It orchestrates the chain **/pwg-live-gate → exact preflight → /pwg-bounded-run → /pwg-window-close**: gate the profile, pick the lane head, cost-gate the keys, spend a bounded headless window, then close it. The **execution route is the headless CLI (manifest v2)** — the Workflow-from-session run route is retired ([H1110](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1110-Opus_SanskritLexicography_pwg-ru-post-h1080-audit-fix-skills-c4-restart_17.07.26.md) — Opus 4.8, PWG-RU post-H1080 audit fix skills + c4 restart); a Workflow session is forensics-only, never production. Working dir: `~/Documents/GitHub/SanskritLexicography/RussianTranslation`. Generation tier = **Sonnet 5** (`claude-sonnet-5`) — resolve the alias to the exact version from the run environment; the deterministic Python gates do the QA (no per-card LLM judge).

Orientation (read before touching pwg_ru code on a fresh session): [PIPELINE_HISTORY.md](https://github.com/gasyoun/SanskritLexicography/blob/master/RussianTranslation/PIPELINE_HISTORY.md) (chronological "how did we get here" — major fixes, recurring failures, current state). Live operating procedure: [RUN_FREQ_MAX.md](https://github.com/gasyoun/SanskritLexicography/blob/master/RussianTranslation/src/pilot/RUN_FREQ_MAX.md) (worked `vid` example with real numbers). Current scale state: [SCALE_READINESS_CENSUS_2026-07-12.md](https://github.com/gasyoun/SanskritLexicography/blob/master/RussianTranslation/SCALE_READINESS_CENSUS_2026-07-12.md).

---

## 🚦 Gate 0 — live-readiness via /pwg-live-gate (before spending ANY tokens)

Before generating anything, run **/pwg-live-gate** (read its file / follow it) for the target profile (bootstrap was c4-only at H1110 (Opus 4.8); the live slot is whatever /pwg-live-gate Step 0 rules — **c4 RETIRED, c1 sole live lane since 19-08-2026**). It fires one representative ≥5 KB health call, then one separate `dq_canary_puregloss` synthetic-control call through the headless CLI, and **derives GO/NO-GO mechanically from a named policy** — health PASS = the measured wall latency strictly under the wall ceiling AND `duration_api_ms` strictly under the route ceiling AND zero connection errors, **all derived from `probe_log.POLICIES[probe_log.CURRENT_POLICY]` at run time — never a number restated in prose here** (this line once said "< 30 s" and stayed stale across two policy generations; the probe prints the ceiling it judged by on its own `ceiling` header line — read that); canary PASS = 3/3 senses + zero SAN-LOSS/TNMASK/unmapped/schema failures. Do **not** hand-assert GO: the gate rejects any supplied GO that contradicts its `gate_reason` or measured result.

The gate replaces the old single-session Workflow probe *and* the standing "the host is degraded, paid windows are banned" assertion — degradation is now **measured per run** by the gate, not asserted from a fixed date. The code no longer falls (H189 (Opus 4.8)/H220 (Sonnet 5)/H304 (Fable 5) hold); the gate reads whatever the *environment* is right now.

**/pwg-live-gate NO-GO → STOP the drain — but there is 0-token work to do instead** (all runnable while the host is degraded, none spends a generation token):
- the **H809 W1** verb-lane rootmap side-lane (`ROOT_SPLIT_MIN=0` → `src/verify_root_glue.py` hard gate → reset the env var);
- **free TM promotions** — roots with `agent_expected_after_tm == 0` promote off the TM with no live call (G9);
- the **completeness re-audit sweep** (G10: `ru_coverage.py --min 100 --strict-denominator` + `lang_parity_check.py`), and re-auditing already-promoted roots through the current fixed gates;
- **/pwg-review-packet** on the ~11.5k already-promoted `ai_translated` rows (the real throughput bottleneck — G5/G6 sit at 0).

On NO-GO the gate records its **named stop state** and **no paid translation call is made**; the manifest-bound lease stays intact and resumes via `/pwg-drain` **only after a NEW representative health reading passes** (a stale/prior GO does not authorize resumption). The gate persists its own verdict row; log the drain outcome regardless.

---

## Which lane is live (H151 is LEGACY)

> **⚠️ The standing H151 verb batch is MG-ruled LEGACY (05-07-2026) — do not pick it up as "the batch".** This skill is the MECHANISM; the [H151](https://github.com/gasyoun/Uprava/blob/main/handoffs/H151-OxAlpha_RussianTranslation_pwg_ru_verb_batch_drain_04.07.26.md) (OxAlpha — PWG RU verb batch drain) *framing* is retired. Pick the live lane from the 12-07 census / [H809 (Opus 4.8) — PWG→RU scale-unblock fixes](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H809-Opus_RussianTranslation_pwg-ru-scale-unblock-fixes_12.07.26.md):

- **verb drain** — rootmap-gated: only **14 of 701** remaining roots are runnable, **687 lack rootmaps**. The unblock (H809 W1, deterministic 0-token side-lane, runnable *while the API is degraded*): `src/_pilot_gen_merged.py --root-split` writes a rootmap only for GIANT roots (`MIN_SPLIT=8`), so **set `ROOT_SPLIT_MIN=0`** to cover the 513 non-giant roots → `src/verify_root_glue.py` **HARD GATE** → **reset the env var** (leaving it set makes a later `--manifest freq --root-split` split every nominal). Head: `python src\pilot\verb_worklist.py --top 20`.
- **no-PWG** ([H255](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H255-Sonnet_RussianTranslation_pwg_ru_no_pwg_lane_scale_06.07.26.md) Sonnet 5 — PWG RU no-PWG lane scale) — next free index = **6** (`.ai_state`'s `--start-index 4` is stale; `w04`/`w05` already ran; `--start-index` is a pure label knob, content = store-dedup). Planner: `python src\pilot\no_pwg_scale_plan.py`. **Window-name-reuse footgun:** reusing a window label the planner already emitted (the `no_pwg_w03` collision) cross-contaminates ledgers/wf_output — always take the next free index, never re-label.
- **nominal core** ([H179](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H179-Opus_RussianTranslation_pwg_ru_nominal_core_queue_reorder_05.07.26.md) Opus 4.8 — PWG RU nominal core queue reorder, Step 3) — head: `python src\pilot\nominals_worklist.py`.
- **medium50** — **PAUSED, do-not-relaunch-blind.** Split-pool #311 is on master but the prepared canary harness is gone; regenerate the 12 keys and fingerprint-verify `MAX_TRANSLATE=34` / `MAX_HEAL=125` before spending a token.

---

## ⚠️ Launch-phase locks — do not re-litigate

- **Cost gate (cap-and-defer, MG-LOCKED 07-07):** `python src\pilot\perf_preflight.py <root> --json` → `cost_partition.run_now` vs `cost_partition.defer_monster`; `--refuse-over-cost` refuses over the H189 ceiling (per-card $2.00). **`deferred_monsters.jsonl` DOES exist and is a deliberate ledger** ([H2173](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H2173-Opus_SanskritLexicography_pwg-audit-tail-g5-g8-g9-g10_02.08.26.md) G9, Opus 4.8 — PWG audit tail G5/G8/G9/G10; this line previously asserted the opposite): [window_common.py](https://github.com/gasyoun/SanskritLexicography/blob/master/RussianTranslation/src/pilot/window_common.py) defines `DEFERRED_MONSTERS` and its `defer_monster()` appends one `pwg.deferred_monsters.v1` row per cap-and-defer decision, **deduped on (target, reason, UTC day)** so a daily coordinator sweep cannot flood it. It is best-effort by design — a failed write warns and returns `None` rather than breaking a claim/prepare — so **an absent row is not proof no defer happened**, and an empty file is a legitimate state. Read the ledger as the durable defer history; read the `cost_partition` keys as the live partition for the root in front of you. `defer_monster` / `defer-calibrate` roots (e.g. `sTA` = 241 agents) get their own dedicated human-budgeted session — never squeezed onto a tail. ⚠️ `parse_workflow_cost.py`'s rate table is still Sonnet-4.x (H809 W2) — **token counts/ratios are trustworthy, absolute $ is not** until the table is refreshed.
- **Never run a window of `kAla`-class monsters as bulk** — the gate flags `pril10`'s top-8 heads at ~$4/card even post-fix; the amortization fix cut the presplit lane ~2.5× but does not make monsters cheap. First nominal window uses TYPICAL heads, not top-8-by-size.
- Standing locks: opt2 harness canonical, **one card per translate call — now the structural generator default, no flag needed** (H4054 landed the H2152 ruling as `OUTPUT_BUDGET = 1`; see the call-shape note below), `--tm=auto` on first pass, **≤3-wide concurrency** (Slice-D 18× collapse = 117 transient nulls, the counter-example) — but **bootstrap /pwg-bounded-run is max-wide=1**, tighter than this ceiling. PWG sense order preserved, lean-TR rejected. `csl-orig/v02/pwg/pwg.txt` is **read-only** always.

> **Call shape — ONE card per call, now structural (H2152 ruled 02-08-2026, executed by Opus 5 1M (`claude-opus-5[1m]`); landed as code by H4054 on 04-09-2026).**
> The two live commands in this chain used to disagree: [/pwg-bounded-run](https://github.com/gasyoun/claude-config/blob/main/commands/pwg-bounded-run.md) said "**Call shape stays ONE card per call (`--output-budget=1`); do not batch to save money**" while the generator kept `OUTPUT_BUDGET = 90` as its default and this file carried `90` as a standing lock until 03-09-2026. Batching is decided at *generation* time, so a manifest emitted at budget `90` reaches /pwg-bounded-run already batched and nothing downstream can un-batch it.
>
> The audit is explicit: option **A. one card per call** is the recommendation; option **B. multi-card batch (`OUTPUT_BUDGET=90`)** is a **No** — on the 02-08 run successes climbed 120.4 → 132.0 → 134.5 → **164.3 s**, i.e. 67 % → **91 %** of the per-call budget, so any call above the median dies. Memo: [AUDIT_C4_CALL_SHAPE_QUOTA_VS_WALLCLOCK_02.08.2026.md](https://github.com/gasyoun/SanskritLexicography/blob/master/RussianTranslation/pwg_ru/h2152/AUDIT_C4_CALL_SHAPE_QUOTA_VS_WALLCLOCK_02.08.2026.md).
>
> **H4054 (04-09-2026) made the safe shape structural**: [gen_opt_harness2.py](https://github.com/gasyoun/SanskritLexicography/blob/master/RussianTranslation/src/pilot/gen_opt_harness2.py) now defaults to `OUTPUT_BUDGET = 1` — a **no-flag** production preparation emits one original card per translate call, and the manifest's `batches` each carry exactly one key. The one-card flag still works and may be passed for clarity, but the shape no longer depends on whoever types the command. Deliberate multi-card packing survives only as an explicit experiment/calibration lane (`--output-budget=90` reproduces the 2026-07-03 calibrated packing; `calibrate_perf_harness.py` arms pass it explicitly; legacy byte mode stays reachable via `--budget=N` / `--output-budget=off`). Presplit/heal routing is independent of the batch budget (`PRESPLIT_SOLO_CITE_FLOOR=40`, `SENSE_PRESPLIT_BUDGET=20`, `SELFHEAL_GROUP_BUDGET=12`), so the one-card default does not push ordinary cards into the heal lane (H255/H823/H2160). A lane that deliberately wants batching says so in its own handoff — it is never inherited from a default.

- The three **close-phase** guardrails (capture-from-manifest-output · `--no-tm`-on-requeue · better-attempt-wins) live canonically in **/pwg-window-close** — they fire after the window returns.

---

## Goal-based invocation (G7–G10)

Four `/goal` loops over this same mechanism, spec'd in [GOALS_MANUAL.md](https://github.com/gasyoun/Uprava/blob/main/GOALS_MANUAL.md) § Tier 1. Use them when the stop condition is worth handing to an evaluator. Generation = Sonnet 5; QA = deterministic Python gates.

**G7 · Requeue sub-loop → empty** (runs inside /pwg-window-close Phase 3)
```
/goal drive <root>'s requeue.keys.txt to empty (or clearly diminishing), keeping the better prior attempt on any regression, stop after 3 requeue rounds
```

**G8 · Per-root cap-and-defer**
```
/goal run only <root>'s under-ceiling cards (perf_preflight cost_partition.run_now) and hand cost_partition.defer_monster keys to the human-budgeted lane, stop after 1 preflight + 1 launch
```

**G9 · Session token budget (max-wide=1 bootstrap)**
```
/goal drain queued roots until the session token budget is spent or the lane worklist REMAINING hits 0, one profile / max-wide=1, zero-agent roots first, stop at budget
```
Zero-agent roots (`agent_expected_after_tm == 0`) are free TM promotions — drain first at no live spend.

**G10 · Completeness re-audit sweep** (0-token, safe while the API is degraded)
```
/goal ru_coverage.py --min 100 exits 0 (every root at threshold RU%) and lang_parity_check.py is clean, stop after 1 report + 1 catch-up round per flagged root
```

---

## Phase 0 — Live gate, branch, lane head

```powershell
# Gate 0 first — run /pwg-live-gate; it must return GO for the target profile, or STOP (see above)
cd ~/Documents/GitHub/SanskritLexicography/RussianTranslation
git fetch
git worktree add -b drain/<root-or-label> ..\SanskritLexicography-<slug> origin/master   # main tree is commit-guarded; work in a worktree
python src\pilot\verb_worklist.py --top 20         # or nominals_worklist.py / no_pwg_scale_plan.py for the live lane
python src\pilot\perf_preflight.py <next ~8 roots> # low-agent first; zero-agent = free TM promotions; defer giants
```

`verb_worklist.py --top` prints runnable roots only; missing-rootmap roots show as `blocked_missing_rootmap` (unblock via the H809 W1 `ROOT_SPLIT_MIN=0` side-lane above). Trust live `perf_preflight.py` over old prose estimates. `defer-calibrate` roots get their own session.

## Phase 1 — Launch one root, then close it

1. **Preflight** — `python src\pilot\root_window_status.py <root>` then `python src\pilot\perf_preflight.py <root> --refuse-over-cost`. Trust the command output + `src\pilot\output\window_status.json` over any stale note. Selected keys must pass exact preflight and stay within budget (G8).
2. **Generate** — `python src\pilot\gen_opt_harness2.py <root>` (opt2 canonical; `--tm=auto`, selfheal + binary-split + presplit routing on). **No flag needed for call shape**: the generator default is `OUTPUT_BUDGET = 1` (one original card per translate call) since H4054 landed the H2152 ruling as code — verify from the emitted manifest's `meta.batches` (each carries exactly one key), not from prose. Multi-card packing is an explicit experiment lane only (`--output-budget=90`): see the call-shape note under *Launch-phase locks*. Emits the **manifest v2** (`execution_route: claude-cli-headless`, `profile_slot`, `config_dir_fingerprint`, exact model) that /pwg-bounded-run consumes.
3. **Run (bounded, headless)** — hand the prepared lease / nominal manifest-v2 to **/pwg-bounded-run** (read its file / follow it): headless CLI, **one profile**, `--only-profile <slot> --timeout 600`, `--stop-before-promote`. (**No `--max-agents` on a multi-key window** — it is a total spawn ceiling, not a concurrency width; `--max-agents 1` starves the window and cost H3627 a launch — the same defect [#223](https://github.com/gasyoun/claude-config/pull/223) removed from /pwg-bounded-run. **`--timeout` from code, never prose** — `HARD_TIMEOUT_MS` is single-sourced in `execution_contract.PRODUCTION_HARD_TIMEOUT_MS` = **600 000** since 07-08-2026 (300 000 on 02-08, 180 000 before), so passing an old figure re-pins a retired per-call ceiling by hand: [#983](https://github.com/gasyoun/SanskritLexicography/issues/983), H2173 G9, [H2254](https://github.com/gasyoun/SanskritLexicography/blob/master/RussianTranslation/pwg_ru/h2254/BOUNDED_300S_CEILING_CONVERGENCE_AND_LIVE_PROOF_07-08-2026.md).) Bootstrap forbids retries/requeues/replacements/widening. **Never** a Workflow-from-session run and never a manual Max save — that route is retired.
4. **Close** — hand the completed window (the headless manifest-v2 output + coordinator lease/current-attempt artifacts) to **/pwg-window-close** (read its file / follow it): validate audit + provenance + final schema + exact key delta → requeue `--no-tm` (better-attempt-wins) → store-backup → promote via `src\promote_final_cards.py` + one-time TM rebuild → ledger row → micro-commit. Loop back to Phase 1 for the next queued root until the session budget or the lane REMAINING hits 0.

## Phase 2 — Close the session

- Update [`.ai_state.md`](https://github.com/gasyoun/SanskritLexicography/blob/master/RussianTranslation/.ai_state.md): roots done, card counts, gate/probe state, **next lane head**. `.ai_state.md` is contended by concurrent sessions — pull the branch base before editing; on conflict merge additively.
- Open the PR from `drain/<label>` (per-root summary, provenance block with exact model versions + manifest-v2 profile/fingerprint, full blob URLs); enable auto-merge + delete source branch — own repo, no per-PR ask.
- Refresh the [`Uprava/GTD_NEXT_ACTIONS.md`](https://github.com/gasyoun/Uprava/blob/main/GTD_NEXT_ACTIONS.md) resume pointer with the concrete next command, and mirror any human/@DECIDE item surfaced.
- When a batch of roots is promoted + clean, assemble the G5/G6 human gate with **/pwg-review-packet** (the promoted cards are `ai_translated`, not `approved`).
- Report: roots drained, cards generated / clean / requeued / promoted, token/time notes, the **/pwg-live-gate** verdict, the exact resume command.

**Serial multi-profile operation** is documented as an **optional later mode** only — assess profiles one at a time via /pwg-live-gate, each with its own `config_dir_fingerprint`; H1110's (Opus 4.8) bootstrap is c4-only, single-profile, max-wide=1.

**Resume line while the API is degraded** (the runnable 0-token work — the drain itself resumes via `/pwg-drain` once /pwg-live-gate returns GO):
```
Read ~/Documents/GitHub/Uprava/handoffs/H809-Opus_RussianTranslation_pwg-ru-scale-unblock-fixes_12.07.26.md and execute it.
```
Open it in a **Sonnet 5** chat with cwd `~/Documents/GitHub/SanskritLexicography/RussianTranslation`.

_Dr. Mārcis Gasūns_
