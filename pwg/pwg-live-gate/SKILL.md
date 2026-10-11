---
name: "pwg-live-gate"
description: 'PWG-RU live-readiness gate before any paid run: one health call + separate dq_canary_puregloss control via headless CLI; GO/NO-GO derived mechanically from probe_log.POLICIES ceilings; slot auto-resolves (c1). Trigger: "is c1 live". Do not use to run a translation window — this only produces the GO/NO-GO gate; run /pwg-bounded-run or /pwg-drain after a GO.'
---

# pwg-live-gate

Use this skill when the user asks to run the migrated source command `pwg-live-gate`.

## Command Template


# /pwg-live-gate

_Created: 18-07-2026 · Last updated: 01-10-2026_

This skill is the **live-readiness gate** that must return **GO** before **/pwg-bounded-run** spends a paid token. Both readings go through the **headless CLI (manifest v2)** production route — never a Workflow session — and the verdict is **derived mechanically from a named policy — never asserted**. Working dir: `~/Documents/GitHub/SanskritLexicography/RussianTranslation`. The canary's live executor tier is the exact model pinned in the sealed manifest (resolve the alias to its version from the run environment and record it).

Orientation: [PIPELINE_HISTORY.md](https://github.com/gasyoun/SanskritLexicography/blob/master/RussianTranslation/PIPELINE_HISTORY.md) · c4 single-profile gate-0 history [H963_C4_SINGLE_PROFILE_GATE0_HEALTH_2026-07-16.md](https://github.com/gasyoun/SanskritLexicography/blob/master/RussianTranslation/pwg_ru/h963/H963_C4_SINGLE_PROFILE_GATE0_HEALTH_2026-07-16.md) · canary curation [H994_DQ_SANLOSS_CANARY_CURATION_2026-07-15.md](https://github.com/gasyoun/SanskritLexicography/blob/master/RussianTranslation/pwg_ru/h994/H994_DQ_SANLOSS_CANARY_CURATION_2026-07-15.md).

## Step 0 — resolve the profile slot (never assume the script default)

The slot is an argument (`/pwg-live-gate [slot]`). Resolve it in this order:

1. An explicit argument always wins.
2. Otherwise read the current live-lane ruling off the repo itself — the `.ai_state.md` head ([.ai_state.md](https://github.com/gasyoun/SanskritLexicography/blob/master/RussianTranslation/.ai_state.md)) plus the newest `pwg_ru/h*/` gate packet. **As of 21-08-2026 that reads: c4 is RETIRED (MG ruling 19-08-2026), c1 is the sole live lane.** c5 has quota, but its only health reading ([c5 packet 25-07-2026](https://github.com/gasyoun/SanskritLexicography/blob/master/RussianTranslation/pwg_ru/h858/H858_C5_LIVE_GATE_HEALTH_NOGO_2026-07-25.md)) ran ~2× the **then-30 000 ms `production_v1`** ceiling and has never been measured under `production_v3` — swapping profiles is not a proven unblock. Verify no newer row supersedes it before trusting it.
3. If step 2 is ambiguous or stale, demand an explicit slot. Never proceed on a guess.

**Omitting `--account` in Step 1 probes the script's built-in default (`ACCOUNT = "c4"`) — i.e. the wrong, retired lane** ([h963_c4_gate0_probe.py](https://github.com/gasyoun/SanskritLexicography/blob/master/RussianTranslation/src/pilot/h963_c4_gate0_probe.py)). Every command below takes `<slot>`; its config dir on Windows is `<claude-tools>\profiles\claude<N>\.claude` (c1 → claude1, c4 → claude4, …). On macOS there is no `<claude-tools>` volume: `<CONFIG_DIR>` below = the slot profile dir actually provisioned on this machine (read it from the slot's own manifest/`.ai_state` row; if no `<slot>` profile is provisioned on the Mac, the gate is box-gated to the Windows profiles machine — say so, do not guess a path).

## Constants are DERIVED — read them from code and probe output, never from prose (including this file)

Every numeral below is dated convenience only; the derivation is the contract.

| When | What changed | Source of truth |
|---|---|---|
| 31-07-2026 | wall ceiling 30 000 → 65 000 (`production_v2`); measured-only, warm-up advisory | `probe_log.POLICIES` |
| 02-08-2026 | WALL gates completion (H2160 (Opus 5) option A); `production_v3` = 80 000 wall + 45 000 route, both derived from the 8-reading c4 series (pass 2/8 → 5/8); between-sittings ration minted (H2174 (Opus 5)) | [H2138](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H2138-Opus_RussianTranslation_probe-ceiling-paired-readings-946_01.08.26.md) (Opus 5) |
| 02-08-2026 | `HARD_TIMEOUT_MS` 180 000 → 300 000 | H2173 G9 (Opus 4.8) |
| 07-08-2026 | `HARD_TIMEOUT_MS` 300 000 → 600 000; above-ceiling requests now RAISE pre-spawn instead of clamping; constant single-sourced in `execution_contract.PRODUCTION_HARD_TIMEOUT_MS` | [H2254 convergence doc](https://github.com/gasyoun/SanskritLexicography/blob/master/RussianTranslation/pwg_ru/h2254/BOUNDED_300S_CEILING_CONVERGENCE_AND_LIVE_PROOF_07-08-2026.md), [#983](https://github.com/gasyoun/SanskritLexicography/issues/983) |
| 19-08-2026 | refusal ≠ malformed_output split (`StructuredRefusal`, `refusal_excerpt`); health-probe prompt shares the production task shape (`MASK_PREAMBLE`); raw envelope captured on validation failure | [PR #1837](https://github.com/gasyoun/SanskritLexicography/pull/1837), [H3157](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H3157-Opus_SanskritLexicography_h3144-residual-c1-paid-window-measurement_19.08.26.md) (Opus 5) |
| 19-08-2026 | **c4 retired; c1 sole live lane** (c5 quota, last measured ~2× the then-30k `production_v1` ceiling on 25-07 — no `production_v3` reading exists) | MG ruling, `.ai_state.md` head + [c5 packet](https://github.com/gasyoun/SanskritLexicography/blob/master/RussianTranslation/pwg_ru/h858/H858_C5_LIVE_GATE_HEALTH_NOGO_2026-07-25.md) |
| 21-08-2026 | c1 NO-GO on the weekly Pro cap (reset 23-08 14:00 Europe/Moscow); an expired receipt authorizes nothing | [H3228 packet](https://github.com/gasyoun/SanskritLexicography/blob/master/RussianTranslation/pwg_ru/h3228/H3228_C1_LIVE_GATE_NOGO_21-08-2026.md) |

## Step 1 — health: ONE representative ≥5 KB `<slot>` call (no retry, no reroll inside a sitting)

Windows (PowerShell):

```powershell
cd ~/Documents/GitHub/SanskritLexicography/RussianTranslation
python src\pilot\h963_c4_gate0_probe.py --account <slot>
```

macOS / Linux (sh):

```sh
cd ~/Documents/GitHub/SanskritLexicography/RussianTranslation
python3 src/pilot/h963_c4_gate0_probe.py --account <slot>
```

One fresh dated D-K attempt for `<slot>`. The probe emits a **schema-carrying ≥5 KiB** payload (an undersized payload is `NO-GO — payload undersized`, not a health reading) and prints `GATE-0 VERDICT: PASS` / `NO-GO`.

- **Health PASS policy — derive everything from `probe_log.POLICIES[probe_log.CURRENT_POLICY]`:** the **MEASURED** reading strictly under the policy wall ceiling (via `max_account_orchestrator.PROBE_LATENCY_CEILING_MS`) **AND** its `duration_api_ms` strictly under the policy route ceiling (`api_ceil_ms`) **AND zero connection errors**. The **warm-up is advisory for latency only** — still taken, recorded and reported; a warm-up that ERRORS or is ABSENT still fails the gate. Read the live numbers off the probe's own ceiling header line.
- **Why TWO independent ceilings:** wall = `duration_api_ms` + `api_gap_ms` and the two move independently (measured api/wall ratio 0.25–0.72), so a threshold on the sum cannot express route health. Wall gates *completion* (non-API overhead still kills calls, and a killed call still bills); route gates *health*; neither number may be REUSED as the other's ceiling — v2's 65 000 was fitted to wall when no reading carried `duration_api_ms` at all, and judging api against it would have admitted 100–350 s walls. And the gap is NOT a stable tax: H2160 (Opus 5) measured gaps of 46 492 ms and 192 682 ms after four same-day readings at "stable ~17–23 s".
- **Classify the reading BEFORE interpreting its latency.** Six classes:
  1. success + schema-valid → judge against both ceilings normally.
  2. hang (no return) → a **QUOTA candidate FIRST**, not a latency reading — a throttled CLI does not report `429`, it hangs (FINDINGS §270).
  3. our-kill at exactly `HARD_TIMEOUT_MS` (read the constant; ~600 0xx ms since 07-08; historical clusters at ~300 0xx and 180 04x–23x ms identify kills under the OLD ceilings) — a third class, never filed as hang or latency; `reservation_timeline.py` differences the ledger stamps.
  4. up-front `rate_limit` refusal — returns fast, tiny payload, all token counters 0; if `observed_cost_usd: 0` with `cost_evaluable: true` it is genuinely free, while `cost_evaluable: false` means the zero is NOT evaluable — **read `cost_evaluable` before reporting either zero**.
  5. `classification: 'refusal'` (`StructuredRefusal` — prose present, structured output absent) — a REFUSAL, not `malformed_output`; do not hunt a parser bug that does not exist ([headless_worker.py](https://github.com/gasyoun/SanskritLexicography/blob/master/RussianTranslation/src/pilot/headless_worker.py):1185-1194; the probe half splits identically).
  6. `malformed_output` — present-but-unparseable output; distinct from class 5.
  On any validation failure the raw envelope is captured automatically (`write_failed_envelope` / `_write_probe_raw`) — attach it to every NO-GO report; the H3144 (Opus 5) — h858-residual c1 window after canary NO-GO — diagnosis survived only because of it.
- **Still read and report BOTH latencies on every reading.** ~45 % of a typical wall reading is in-CLI scaffolding, so `api_gap_ms` stays the right diagnostic for why a NO-GO happened — and **never re-read a NO-GO on the friendlier number**: a wall NO-GO whose `duration_api_ms` looks healthy is still a NO-GO. That is the gate working, not a technicality (the 02-08 proof case: the fastest API time ever recorded on c4, 16 445 ms, still failed the then-wall gate on 49 846 ms of scaffolding — one number cannot do both jobs).
- **`claude auth status --json` proves CREDENTIALS, not QUOTA.** It is free and ~1 s — run it, but it returned `loggedIn`/`max` throughout the 31-07 throttle. Never read it as a quota signal.
- **A bare `['claude']` fallback resolution is the D-R tooling defect, NOT a health signal** — NO-GO, fix the shim, do not read it as latency.
- **No retry, no re-warm, no reroll — INSIDE a sitting.** A NO-GO leaves the same immutable trace as a PASS; on NO-GO STOP — no canary, no production window.
- **BETWEEN sittings — the ration (MG ruling 02-08-2026, H2174 (Opus 5) — medium50 presplit live run after health pass).** A NO-GO is evidence about *this hour*, not about the route: the lane is bimodal on a timescale of hours (PASS 43 815 → NO-GO 75 561 → NO-GO 96 520 ms across one day, same profile/prompt/ceiling). So a later attempt is legal under a ration: **≥ 6 h spacing** between probes on a profile, **≤ 2 attempts per UTC day**, and after **3 consecutive NO-GO days** the lane stops and the ceiling goes back to re-derivation ([H2138 (Opus 5) — probe-ceiling paired readings](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H2138-Opus_RussianTranslation_probe-ceiling-paired-readings-946_01.08.26.md)) rather than to another probe. At ~$0.55/probe this caps probe spend at ~$1.10/day. Two hard corollaries:
  - **Arm any scheduled retry off the LAST `probe_call` row ts + spacing + margin, NEVER off the run-id start** — the gap between the two equals the slowest call in the sitting and is therefore largest exactly on the NO-GO days that make a retry worth scheduling (H2259 (Opus 5) — c4 gate attempt-2 record-and-act: a one-shot armed off run-id start fired 362 s early and correctly refused).
  - **Read the ration as a SPEND CAP, not a route to a green gate.** When readings sit above a correctly derived ceiling (the worked example: median 76 988 vs the then-live 65 000, pass 2/8), the fix is FITTING the ceiling from the measured distribution — never raising one to pass a gate, which stays banned. That fitting produced `production_v3`; the situation is resolved, the numbers stay as the example.

If health is NO-GO, emit the verdict (Step 3) as **NO-GO** and stop. There is 0-token work available (see /pwg-drain Gate-0), but this gate returns NO-GO.

## Step 2 — canary: ONE separate dq_canary_puregloss synthetic-control call

Only if Step 1 is PASS. Run the curated synthetic control through the headless CLI (the production route), never a Workflow session. **The manifest has a committed builder — never hand-author one (H2174/H2245, both Opus 5).** Step 2a is offline and spends nothing; only 2b is the paid call.

Windows (PowerShell):

```powershell
# 2a — build manifest v2 + real lane preflight. ZERO paid calls. Prints manifest /
# harness / preflight / sha256 AND the paste-ready 2b line, including the currently-
# correct --timeout derived from execution_contract.PRODUCTION_HARD_TIMEOUT_MS.
python src\pilot\canary_manifest_build.py --profile-slot <slot> `
       --config-dir "<claude-tools>\profiles\claude<N>\.claude" --outdir src\pilot\output\<hid>

# 2b — execute headless (ONE call, one profile). THE ENV VAR IS REQUIRED — nothing else
# supplies it (H3144, Opus 5); set it BEFORE pasting 2a's printed command verbatim.
$env:CLAUDE_CONFIG_DIR = "<claude-tools>\profiles\claude<N>\.claude"
python src\pilot\headless_worker.py <outdir>\execution_manifest.canary.json `
       --output <outdir>\out.canary.json --status-out <outdir>\status.canary.json `
       --only-profile <slot> --max-agents 1 --max-calls 3 `
       --manifest-sha256 <sha from 2a> --preflight <outdir>\preflight.canary.json `
       --call-reservation <outdir>\calls.canary.json --run-id <run-id> `
       # ...plus whatever else 2a's printed line adds (its own --timeout among it)
```

macOS / Linux (sh) — `<CONFIG_DIR>` = the slot profile dir provisioned on this machine (see Step 0):

```sh
# 2a — build manifest v2 + real lane preflight. ZERO paid calls.
python3 src/pilot/canary_manifest_build.py --profile-slot <slot> \
       --config-dir "<CONFIG_DIR>" --outdir src/pilot/output/<hid>

# 2b — execute headless (ONE call, one profile). THE ENV VAR IS REQUIRED (H3144, Opus 5);
# set it BEFORE pasting 2a's printed command verbatim.
export CLAUDE_CONFIG_DIR="<CONFIG_DIR>"
python3 src/pilot/headless_worker.py <outdir>/execution_manifest.canary.json \
       --output <outdir>/out.canary.json --status-out <outdir>/status.canary.json \
       --only-profile <slot> --max-agents 1 --max-calls 3 \
       --manifest-sha256 <sha from 2a> --preflight <outdir>/preflight.canary.json \
       --call-reservation <outdir>/calls.canary.json --run-id <run-id>
       # ...plus whatever else 2a's printed line adds (its own --timeout among it)
```

- **Missing `CLAUDE_CONFIG_DIR` → exit 2 `{"classification": "configuration", ...}`** — a provisioning abort, not a canary reading: nothing spent, no attempt consumed. Fix the env var and rerun 2b; do not log it as a gate result. (The builder's printed command still omits the env prefix — the paste trap H3144 (Opus 5) paid an invocation to find.)
- **`--timeout` comes ONLY from 2a's printed command, never from prose here.** Effective bound is `min(--timeout × 1000, budgets.timeout_ceil_ms, HARD_TIMEOUT_MS)` with `HARD_TIMEOUT_MS = execution_contract.PRODUCTION_HARD_TIMEOUT_MS`; a request above the ceiling raises before any subprocess spawns. See the history table for how this file carried a stale figure after each of the three constant changes — that is exactly what this rule prevents.
- **Judge — the verdict is CODE, not prose (H2159 — Fable 5), and the receipt carries provenance (H2245-era inputs):**

Windows (PowerShell):

```powershell
python src\pilot\canary_gate.py judge <out.canary.json> --receipt <canary_receipt.json> `
    --manifest <sealed manifest> --status <outdir>\status.canary.json `
    --call-reservation <outdir>\calls.canary.json --run-id <run-id>
```

macOS / Linux (sh):

```sh
python3 src/pilot/canary_gate.py judge <out.canary.json> --receipt <canary_receipt.json> \
    --manifest <sealed manifest> --status <outdir>/status.canary.json \
    --call-reservation <outdir>/calls.canary.json --run-id <run-id>
```

  GO/NO-GO is computed mechanically (synthetic-key check via the promote lane's `SYNTHETIC_KEY_RE`, 3/3 senses with Russian content, zero `{Tn}`/SAN-LOSS/UNMAPPED, profile recorded). An input you do not pass is recorded `null`, never zero — the 05-08-vs-06-08 distinction (`observed_cost_usd: 0` meaning "not evaluable" versus a genuine $0 refusal) is what a fabricated zero would erase, turning a cost FLOOR into a total.
- **Canary PASS policy:** the returned card carries **3/3 senses** and **zero** SAN-LOSS / TNMASK / unmapped-token / schema failures — the same deterministic gates real cards use (`ru_coverage.py`, `lang_parity_check.py`, sense-count, TNMASK).
- **Synthetic output is NEVER promoted or retried.** `dq_canary_puregloss` is `synthetic_control` provenance; the promoter refuses it (C-05) — that refusal is expected, not a failure. The canary is billed (a real paid call) but must never reach the store.
- **Refusal-class canaries (H3157, Opus 5): a SECOND refusal-class canary on the same profile is a STOP and a diagnosis task — never a third attempt.**

## Step 3 — mechanical verdict (a named policy, not an assertion)

```
gate_reason  = HEALTH_NOGO            if health != PASS
             = CANARY_FAIL(<reasons>) if canary  != PASS   (senses<3 | SAN-LOSS | TNMASK | unmapped | schema)
             = LIVE_GO                if health PASS AND canary 3/3 clean
verdict      = GO   iff gate_reason == LIVE_GO
             = NO-GO otherwise
```

- Derive `verdict` **only** from `gate_reason` + the two measured results. **Reject any supplied `GO`** (operator, prior note, ambient state) that contradicts `gate_reason` or the measured representative-payload result — a hand-asserted GO over a HEALTH_NOGO/CANARY_FAIL is a hard error, not an override.
- Record the verdict with both latencies, connection-error counts, the classification class, canary sense/flag counts, and the full manifest-v2 provenance block (profile slot · fingerprint · route · model version · validation method). Persist to the `<slot>` gate log — this row is what /pwg-bounded-run consumes.

## Hand-off

- **GO** → the profile is live-gated; proceed to **/pwg-bounded-run** with the same `--only-profile` and the recorded manifest-v2 provenance.
- **NO-GO** → record the named stop state (`HEALTH_NOGO` / `CANARY_FAIL(<reasons>)`) to the `<slot>` gate log and **make no paid translation call** — not a canary rerun, not a bounded window, nothing billed. The manifest-bound lease is left intact and unconsumed. **Resume the same manifest-bound lease by rerunning /pwg-drain only after a NEW representative health reading passes** — a fresh ≥5 KB `<slot>` health call (Step 1) returning PASS; a stale or prior GO — including an expired receipt — authorizes nothing. Report `gate_reason`, both readings, and that exact resume condition.
## References

Detail moved verbatim (read on demand — lives in the rendered skill copy's references/):

- `references/canary-policy-detail.md` — Canary spend/identity details — ≤6h GO receipt is the spend key; manifest bound by live config_dir_fingerprint + known-good artifacts
