---
name: "model-provenance-audit"
description: 'Sweep run logs, handoffs, .ai_state.md, workflow scripts for bare model tiers or unresolved aliases and upgrade them to tier + exact version ("Sonnet 5 (claude-sonnet-5)"). Triggers: check model provenance, before publishing handoff/paper methods. Do not use to onboard or qualify a new model — that is /model-lane-onboard or /model-router-canary; this only fixes bare-tier mentions in existing text.'
---

# model-provenance-audit

Use this skill when the user asks to run the migrated source command `model-provenance-audit`.

## Command Template


# /model-provenance-audit

_Created: 03-07-2026 · Last updated: 05-09-2026_

## Default path (seconds — run this first)

From the Uprava clone root:

```sh
python tools/model_provenance_audit.py <path>
```

LLM / judgment residue only after the script report exists (or the script exits non-zero with a real defect).

Standing rule: a tier without a version is a defect — "Sonnet" is ambiguous a month later. This sweep finds and fixes every under-specified model attribution in a target path.

Argument: a repo, directory, or file to sweep (default: the current repo's authored/provenance MD + run scripts).

## Phase 1 — Find candidates

Run the canonical script — do NOT grep the target by hand (same invocation as the
default path above):

[`tools/model_provenance_audit.py`](https://github.com/gasyoun/Uprava/blob/main/tools/model_provenance_audit.py) finds every candidate site (report-only; Phases 2–4 remain in-model because alias→version resolution needs git-blame date correlation). It covers, in `.md`, `.ai_state.md`, run logs, and workflow/harness scripts:

- Bare tier names near attribution context: `(Sonnet|Opus|Haiku|Fable)(?!\s*\d)` — filter hits to lines about who/what ran a step (skip prose about the products generally).
- Unresolved aliases in code: `model\s*[:=]\s*'"['"]` and `--model (sonnet|opus|haiku)`.
- Version-less API IDs: `claude-(sonnet|opus|haiku)(?!-\d)`.

## Phase 2 — Resolve alias → version

- For scripts: read the **actual** `model:` values in the script — never trust a harness description or comment.
- Resolve via the mapping current **at the time of the run** (git blame the log/handoff for its date). Today's mapping: Fable 5 = `claude-fable-5`, Opus 4.8 = `claude-opus-4-8`, Sonnet 5 = `claude-sonnet-5`, Haiku 4.5 = `claude-haiku-4-5-20251001`. Prior versions to resolve against older-dated runs: Sonnet 4.6 = `claude-sonnet-4-6`.
- If the run predates a mapping you can establish, or the alias genuinely can't be resolved (harness gone, log silent), mark it `(version unresolved — alias '<x>' at <date>)` rather than guessing.

## Phase 3 — Fix in place

Rewrite each attribution as `Tier X.Y (\`exact-model-id\`)`. In multi-step/workflow provenance, every step gets its own tier+version — generation vs judge vs orchestration may differ. Document failed/aborted steps' models too.

## Phase 4 — Report + commit

Commit per repo (`docs:` prefix). Report: fixed count, unresolved count with locations, and any script whose actual `model:` contradicted its surrounding documentation (those are the dangerous ones — say so explicitly).

_Dr. Mārcis Gasūns_
