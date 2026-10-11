---
name: "tier-audit"
description: 'Re-tier a specified handoff batch by economics and difficulty. Use only when explicitly invoked as /tier-audit.'
---

# tier-audit

Use this skill when the user asks to run the migrated source command `tier-audit`.

## Command Template


# /tier-audit

_Created: 17-07-2026 · Last updated: 13-09-2026_

The [FINDINGS §90](https://github.com/gasyoun/Uprava/blob/main/FINDINGS.md) correction, made a standing discipline: a 10-handoff batch was once minted on the premise that Fable = a cheap bulk lane, when the model card says the opposite (Fable 5 (`claude-fable-5`) is the most capable AND most expensive tier, ~2× Opus 4.8). Only 3 of 10 survived re-tiering. The root trap: Opus/Sonnet/Haiku are size-tier names, but "Fable" is a specialty name that signals nothing about price/capability — so tier is READ from the model card, never inferred from the name, prior usage patterns, or vibes.

Argument: the batch to audit (H### range, a mint session's output, or "all 🟡 queued for <tier>"); defaults to the most recent `--batch` mint.

Evidence base: [FINDINGS §90](https://github.com/gasyoun/Uprava/blob/main/FINDINGS.md) (H966–H975: 10 minted → 7 re-tiered) + the ×2-repeated user ask Q2607-59. Dedupe: `/skill-fit-audit` audits skill-vs-task fit; this skill audits MODEL-vs-task fit across a batch.

## Phase 0 — Load ground truth (never skip, never cache)

Read the CURRENT model table from the relevant provider source — `claude-api`
(`shared/models.md`) for Claude and the current Codex/OpenAI environment or
official model documentation for Codex. Prices and capabilities change;
hardcoding them recreates the §90 error. Record provider, model id, capability
evidence, and cost evidence in the audit. The current Codex identity is
`gpt-5.6-sol`, but its permanent filename route is `Codex`, never `Sol`.

## Phase 1 — Inventory the batch

Per handoff: filename `<Model>` token (the HARD gate — `next_task_scan.py` reads it off the filename, no registry note overrides it), the body `Model:`/intended-executor line, and the task's actual content.

## Phase 2 — Classify difficulty

Rule each handoff into a lane, one reasoning line each:

- **Mechanical/templated** (sweeps, fills, renames, format conversions, fan-outs with a fixed recipe) → Haiku 4.5 (`claude-haiku-4-5`) / Sonnet 5 (`claude-sonnet-5`).
- **Analytical** (corpus probes, stats, structured audits with clear criteria) → Sonnet 5 (`claude-sonnet-5`).
- **Design/judgment** (phase design, dedupe calls, architecture, hostile review) → Opus 4.8 (`claude-opus-4-8`).
- **Premium judgment** (register-sensitive writing, adjudication rulings, eval design, long-horizon agentic work) → Fable 5 (`claude-fable-5`) — a scarce resource, assigned only where the cheaper tier demonstrably degrades the output.
- **Codex-native execution** (repository-scale implementation, tool-driven
  repair, or a handoff explicitly marked `needs Codex`) → `Codex`, with exact
  current model `gpt-5.6-sol` in the body. This is a provider route, not a
  cheaper/more-expensive point on the Claude tier ladder.

The burden of proof sits on the EXPENSIVE tier: "the org used Fable for this before" is pattern-matching, not evidence (§90's reinforcement trap).

## Phase 3 — Re-tier the mismatches

For each mismatch, in ONE commit per handoff (or one batch commit in a worktree for Uprava-registry repos):

1. `git mv` the file to the corrected `<Model>` token — the filename is the enforcement point.
2. Edit the body's intended-executor line to match (tier + exact version) — the `check_handoff_executor_token.py` hook blocks a token/body mismatch, so both move together.
3. If the registry row names a tier, buffer that correction through the [H1342](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1342-Opus_Uprava_bookkeeping-batching-and-wip-cap_19.07.26.md) (Opus 4.8 — Uprava bookkeeping batching & WIP-cap) reconcile path (Uprava `tools/registry_reconcile.py add registry-clause …`, the same buffer [`/handoff-close --reconcile`](https://github.com/gasyoun/claude-config/blob/main/commands/handoff-close.md) writes to) rather than committing `handoffs/README.md` per handoff — a whole batch's row edits then land in ONE commit at `registry_reconcile.py flush` (run for you by /handoff Phase 6 and /gtd-sync Phase 0).

Never re-tier a handoff that is claimed/in-flight (check Active Claims + 🔒 lines first) — coordinate or skip.

## Phase 4 — Report the delta

Persist (registry note, or a dated audit doc for big batches — never chat-only): per-handoff before→after table with the reasoning line, tier×count totals, and the estimated cost delta from the Phase-0 price snapshot. Cite [FINDINGS §90](https://github.com/gasyoun/Uprava/blob/main/FINDINGS.md) as the governing rule; if the audit surfaced a NEW mis-tiering pattern, `/findings-append` it.

_Dr. Mārcis Gasūns_
