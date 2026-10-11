---
name: "model-router-canary"
description: 'Qualify cheap routes behind spend-gated canary: reserve, ticket, served-model attestation per dispatch, GO/durable NO-GO, no reroll. Runs on attested tier: Opus 5 (claude-opus-5). Trigger: "qualify router.cheap". Do not use for a first-time model onboarding — that is /model-lane-onboard; this qualifies a CHEAP ROUTE behind a spend-gated canary.'

---

# model-router-canary

Use this skill when the user asks to run the migrated source command `model-router-canary`.

## Command Template


# /model-router-canary [route | version | H###]

_Created: 12-08-2026 · Last updated: 26-09-2026 (H5504: mechanical route ported)_

## Default path (seconds — run this first)

The probe mechanics are ported to a deterministic script (Uprava `tools/model_router_canary.py`, H5504): urllib dispatch, latency + HTTP status per probe, served-model id read verbatim, baseline diff, report + JSON sidecar — and NO verdict:

```sh
# probe plan it WOULD run (DEFAULT, zero network, zero env reads):
python3 ~/Documents/GitHub/Uprava/tools/model_router_canary.py --config canary.json
# live probes (N dispatches; API key read from the env var named in canary.json, never echoed):
python3 ~/Documents/GitHub/Uprava/tools/model_router_canary.py --config canary.json --baseline canary_prev.json --run --out canary_report.md
# canary.json: {"route":..., "endpoint_env":"KEY_VAR", "model":..., "probes":[...], "n":..., "endpoint":"https://..."}
```

LLM / judgment residue only after the script report exists (H5504) — and the GO/NO-GO verdict is deliberately NOT in the script.

Gate that stays with the LLM: the routing verdict — GO or durable NO-GO on the evidence pack (Phase 0 spend authorisation, served-model attestation read, failure class naming). The script produces measurements; the verdict, and everything Phase 0 forbids before it, are not ported.

Encodes the router-qualification contour that ran **7 times** across the H2323–H2591 window — census candidate **C34** in [SKILL_CANDIDATES_CENSUS_H2323_H2591_12_08_2026.md](https://github.com/gasyoun/Uprava/blob/main/archive/SKILL_CANDIDATES_CENSUS_H2323_H2591_12_08_2026.md), force-built on MG's instruction 12-08-2026 ahead of the sheet vote ([H2596](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H2596-Opus_claude-config_skill-build-c34-model-router-canary_12.08.26.md)).

Argument: the route/profile to qualify, the released version to requalify at, or a single `H###` sitting to execute.

## The one-sentence version

**A cheap route is not qualified because it returned something plausible — it is qualified when you can prove which model answered, on which single dispatch, against a schema that was generated from the same fixture as the prompt, under a spend authorisation issued for this sitting.** Every rule below is one of those clauses, and each was paid for.

## Why it is this paranoid

The lane's most instructive session, [H2539](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H2539-Opus_SanskritLexicography_router-cheap-canary-attested-authorised_10.08.26.md) (Opus — router-cheap canary attested+authorised), spent both authorised calls and returned **NO-GO** — not from a route defect, a model substitution, or a bad translation (the translation was clean, 3/3 senses, 0 defects across 13 checked classes). It failed on a **contradiction between two artifacts written in the same session**: the prompt demanded each sense's German skeleton line verbatim, the frozen schema pinned the gloss alone. The model obeyed the prompt and failed the schema.

The second lesson is worse: **the offline selftest passed 24/24 and could not have caught it**, because it was built from the schema's own constants. A test that derives its fixtures from one side of a contract cannot see that the other side disagrees.

## Phase 0 — Spend authorisation (hard gate, non-inheritable)

**A spent authorisation cannot be inherited, renewed, or reinterpreted.** Each live sitting needs a fresh, explicit human ruling naming the exact number of reservations and dispatches. Until that ruling exists in writing: do not claim the handoff, do not reserve, do not invoke the route, **do not even run a health probe** — a probe is spend.

Record the authorisation link in the handoff's evidence block. A handoff with no fresh ruling is `⛔ BLOCKED`, and that is a legitimate terminal state to hand back.

## Phase 1 — Offline suites, before a single token

Run every offline suite the harness owns — bridge selftest, canary-contract, exact-dispatch attestation, route, pipeline. **Anything short of full PASS is an immediate NO-GO with zero calls.** This is the cheapest possible failure and the only one that costs nothing.

Then apply the H2539 lesson: **prompt and schema must be generated from ONE canonical fixture**, and the regression must extract a test instance *from the prompt itself* and validate it against the *generated* schema. A prompt/schema divergence is invisible to any test built from one side alone.

## Phase 2 — Reserve before spend, seal after return

The bridge is two-phase and crash-safe:

```
prepare-external   -> reserve a ticket BEFORE the call (id, prompt hash, schema hash)
   dispatch        -> exactly one Agent/API call
record-external    -> validate, attest, seal the envelope
```

Invariant at close: **`pending_calls: 0`**. A reservation with no seal means a call may have been spent and lost — that is the crash case the two-phase shape exists for, and it must be recoverable, not guessed at.

## Phase 3 — The two-ticket ladder

1. **Ticket 1 — a tiny capability probe.** Cheap, trivially checkable.
2. **Ticket 2 — the real evaluation ticket**, dispatched **only after Ticket 1's clean PASS.**

`Fail = Ticket 2 after Ticket 1 fails.` The ladder exists so a broken route costs the small call, not the large one.

**No retry. No reroll. No manual repair.** Stop at GO or at a durable NO-GO. Repairing a malformed output by hand converts a measurement into a wish.

## Phase 4 — Attestation: which model, on which dispatch

Two distinct claims, and the second is the one that gets faked by accident:

- **Served ≠ requested.** Record the **exact returned model**, never the one inferred from the picker or profile — the gateway may resolve the generation model differently. `model_matches_request` is a measured field.
- **Bind the attestation to exactly ONE immutable dispatch identifier** (the `tool_use.id` / dispatch id), linking ticket → transcript event/`tool_result` → response wrapper → envelope. Missing, wrong, duplicate, replayed, or window-scoped attestation all **fail**.

The window-scoped trap, stated plainly: attesting over a transcript window proves the served model **for the window**, not **for the single dispatch inside it**. H2539 (Opus, SanskritLexicography router canary) attestation defaulted to sidechain records while the harness logged Agent calls as main turns (101 main / 0 sidechain) and returned `model_matches_request: null` until told to include main turns — which then attested the window, not the call.

**Read call counts from each `tool_result`, never from `tool_use` blocks** — in H2539, 3 `tool_use` blocks backed 2 paid dispatches.

## Phase 5 — Cost honesty

`cost_evaluable: false` is a legitimate, reportable result. **A ledger floor of `0` is not evidence the calls were free.** `Fail = a false zero-cost claim.` If usage counters came back empty, say the cost is unknown and why — an unverified saving is the entire thing this skill exists to prevent someone claiming.

## Phase 6 — Capture the refusal envelope

When a call returns a non-success classification, **persist the raw envelope text** (gitignored, beside the event row) — not just the matched class name.

From [H2326](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H2326-Opus_RussianTranslation_c4-rate-limit-refusal-envelope-capture-1172_06.08.26.md) (Opus — C4 rate-limit refusal-envelope capture): a `rate_limit` classification collapses "account weekly cap" and "per-model capacity refusal" into one word and discards the provider's **reset time**, which is what decides whether the next sitting is worth attempting. On a no-reroll gate rationed to 2 attempts per UTC day, a sitting that yields an unreadable refusal has consumed an attempt and returned nothing diagnosable — the worst available ratio.

## Phase 7 — Verdict, and what a NO-GO forbids

- **GO** — mint the production/rollout handoff, with the qualification report and evidence tree linked.
- **NO-GO** — **do not mint it.** Write the report, name the failure class, and mint (at most) a repair handoff plus a fresh requalification handoff marked blocked on a new spend ruling.
- **Synthetic artifacts must never become promotable** — a canary's output is evidence about the route, never product data.
- Every requalification is a **new sitting**: a version bump (`v1.144.28` → `.32`) invalidates the prior GO and needs its own authorisation.

## Offline-first mode — qualify without a single paid call (U58, vote 04-09-2026)

When the question is «can route R serve pipeline P at all» — not yet «is it good» — run the whole qualification **offline** first, staging the live canary instead of running it. Distilled from [H4057](https://github.com/gasyoun/Uprava/blob/main/handoffs/H4057-OxAlpha_SanskritLexicography_pwg-glm-route-offline-qualification_04.09.26.md) (the GLM-route qualification, sealed-manifest shape) and the deepseek/flash retarget wave that predates this skill ([H2439](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H2439-Sonnet_SanskritLexicography_deepseek-v4-flash-prep-e1-retarget_08.08.26.md) et al.):

1. **Resolved identity receipt.** Resolve the actually-installed model id from config/provider metadata and record it verbatim — local records disagree on slug spellings; the receipt, not the memory, is the identity. Never print secrets while doing it.
2. **Adapter seam only.** Implement the missing provider-adapter seam inside the existing kernel/providers abstraction — reservation ledger, strict request/response schema, raw-response receipts, usage provenance, timeout, serialized bounded calls preserved. No new orchestration framework, no direct SDK calls around the kernel.
3. **Replay receipts, not vibes.** Replay the representative case classes against the adapter with a **zero provider calls** fixture set: pure gloss, source markup, apparatus/homonyms, long card, malformed result, missing-usage. Each case gets a pass/fail receipt with the failure verbatim.
4. **Unknown cost fails closed.** If the new route has no measured price, unknown stays unknown — never borrow another model's price to fill the bound; where a dollar bound is required, fail closed.
5. **Sealed live manifest.** Prepare the capped live qualification manifest + comparison rubric (reuse the quarantine sample where appropriate), seal it, and **stage — do not run** — the exact capped live instructions for a separately authorized sitting. Model-route validity, mechanical fidelity, and independent semantic quality stay separate gates; existing control-plane review and canary requirements are not self-signed away.
6. **No cutover.** Production stays on its current authorized route until real route + quality evidence exists. "Offline adapter works" is a **qualification packet**, never a verdict.

## Tier lock

**The canary must run on the tier whose served-model identity is being attested** — it cannot be delegated down. Offline engineering around it (bridge, contract repair, fixtures) is legitimately a different, cheaper executor; the live dispatch is not.

## Evidence pack (close only with links)

- fresh human authorisation, naming the call count
- clean checkout pinned to the released tag/commit, no contract redesign mid-sitting
- pre-spend suite results (all green, with counts)
- ticket · `tool_result` · response wrapper · attestation · envelope · ledger · transcript-structure evidence
- exact hashes and dispatch identifiers
- explicit **GO / NO-GO** with its failure class

Missing evidence is **INCONCLUSIVE**, never PASS.

## Hard rules

- **No spend without a fresh, explicit, per-sitting authorisation** — including probes.
- **Offline suites all green, or zero calls.**
- **Prompt and schema from one fixture**; the regression validates a prompt-extracted instance against the generated schema.
- **One dispatch id binds the whole chain**; a window attestation is not a call attestation.
- **Record the served model**, never infer it.
- **No retry, no reroll, no manual repair**; Ticket 2 only after Ticket 1 passes.
- **Never claim zero cost from a zero floor.**
- **NO-GO does not mint the production handoff.**

## Not this skill

| Situation | Use instead |
|---|---|
| Offline N-run reproducibility on a gold set | [`/eval-harness`](https://github.com/gasyoun/claude-config/blob/main/commands/eval-harness.md) |
| Tier+version wrong in docs/logs/handoffs | [`/model-provenance-audit`](https://github.com/gasyoun/claude-config/blob/main/commands/model-provenance-audit.md) |
| A validator's thresholds drifted | [`/validator-recalibrate`](https://github.com/gasyoun/claude-config/blob/main/commands/validator-recalibrate.md) |
| Resuming a lane after a usage-limit stop | [`/limit-relaunch`](https://github.com/gasyoun/claude-config/blob/main/commands/limit-relaunch.md) |
| PROCEED/HOLD between a sample stage and bulk spend | [`/gate-judge`](https://github.com/gasyoun/claude-config/blob/main/commands/gate-judge.md) |

_Dr. Mārcis Gasūns_
