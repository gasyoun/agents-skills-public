---
name: "dot-convention-sweep"
description: 'Apply the org colour-dot convention to findings/outcome registries — 🔴/🟠/🟡 severity vs 🔴/🟡/🔵 outcome — with a disambiguating legend, never reusing an existing dot meaning. Triggers: "apply colour dots", "add severity dots".'

---

# dot-convention-sweep

Use this skill when the user asks to run the migrated source command `dot-convention-sweep`.

## Command Template


# /dot-convention-sweep

_Created: 11-07-2026 · Last updated: 05-09-2026 (H4103: mechanical route ported)_

## Default path (seconds — run this first)

```sh
python ~/Documents/GitHub/Uprava/tools/dot_convention_sweep.py --sweep [--root PATH]
python ~/Documents/GitHub/Uprava/tools/dot_convention_sweep.py --target-file PATH --vocabulary severity|outcome [--emit-legend --outcome-red TEXT --outcome-yellow TEXT --outcome-blue TEXT] [--apply]
```

[`tools/dot_convention_sweep.py`](https://github.com/gasyoun/Uprava/blob/main/tools/dot_convention_sweep.py)
(H4103, residue rank 8) covers Phase 2's mandatory collision grep and Phase
3/4's legend-line emit + splice mechanically. `--sweep` reports untreated
candidate files by heuristic; a file classified `ambiguous` (both severity
and outcome signals fire at once) is never auto-suggested — Phase 1's
severity-vs-outcome call on an ambiguous file stays the LLM gate below,
same for composing the concrete per-item-type outcome wording
(`--outcome-red/yellow/blue`) the script refuses to invent.

Two established colour-dot vocabularies exist and must not be conflated:

1. **Severity dots** — grade *the importance of a claim/finding*: 🔴 3 important · 🟠 2 medium · 🟡 1 minor. Canonical in [`SanskritLexicography/FINDINGS.md`](https://github.com/gasyoun/SanskritLexicography/blob/master/FINDINGS.md) and [`Uprava/FINDINGS.md`](https://github.com/gasyoun/Uprava/blob/main/FINDINGS.md); a claim line is prefixed with the dot, `§N`-numbered, followed by `Evidence:`/`Implication:`/blockquoted `Source:`.
2. **Outcome dots** — grade *the result of one work item* (a batch root, a PR, a repo, a queued correction, an adjudication ruling): 🔴 error/unresolved · 🟡 partial/caveat · 🔵 clean/all-ok. First formalized in [H151 (OxAlpha) — RussianTranslation pwg_ru verb batch drain](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H151-OxAlpha_RussianTranslation_pwg_ru_verb_batch_drain_04.07.26.md)'s root drain ledger, then rolled into `/pr-babysit`, `/dirty-tree-sweep`, `/cologne-correction-queue`, `/gold-adjudicate`.

Argument: a target file/skill to convert, or `sweep` to search for untreated candidates org-wide.

## Phase 1 — Classify the target

Read the target and decide which vocabulary fits, or neither:

- Numbered claims with an evidence trail, no per-item lifecycle (findings, audit reports, architecture reviews) → **severity dots**.
- A batch of discrete work items each with a pass/fail/partial result (drain ledgers, sweep reports, queue entries, per-PR/per-repo rows, adjudication rulings) → **outcome dots**.
- Neither (a decision queue, a lifecycle-state registry like the handoffs README's 🔵-in-work/🟡-yet-to-launch/✅-done) → **do not apply either vocabulary**; a third colour meaning already lives there and adding a fourth reading of the same emoji is a net loss, not a gain.

## Phase 2 — Collision check (mandatory, do not skip)

Grep the target file for existing 🔴/🟠/🟡/🔵 usage. If found with a *different* meaning than the one about to be applied (e.g. the handoffs registry's 🔵 = "in work" lifecycle state), do not silently overload it — either pick the other vocabulary if it fits, or proceed but add an explicit legend line stating what the dots mean **in this file specifically**, referencing that a different meaning exists elsewhere so a reader isn't misled if they've seen the other convention.

## Phase 3 — Apply

- **Severity target**: prefix each finding/claim heading with its dot; add one legend line near the top of the findings section naming the 3 tiers and linking the canonical FINDINGS.md files.
- **Outcome target**: add a status column/prefix to the per-item table or list; add one legend line defining 🔴/🟡/🔵 for *this* item type (don't reuse the generic wording verbatim — say what "error", "partial", and "clean" concretely mean for this kind of item, as `/gold-adjudicate` and `/cologne-correction-queue` do).
- Never invent a fourth colour or repurpose 🔵/🟠 across the two vocabularies (outcome dots skip 🟠 entirely — only 3 states: error/partial/clean).

## Phase 4 — Deliver

Repo-tracked files: commit only if the user asked, or ask before committing (standard git safety protocol) — don't bundle it silently into an unrelated commit. `~/.claude/commands/*.md` skill edits: commit + push to the `gasyoun/claude-config` repo directly (config sync is pre-authorized, no PR needed). Repo-tracked docs (FINDINGS-style, handoffs, reviews): branch + PR + auto-merge per normal repo discipline — check the local branch actually tracks the repo's real default branch first (`git remote show origin` for `HEAD branch`); a stale/misnamed local branch tracking a leftover feature branch is a real failure mode here, recover via `/branch-contention-recover` before pushing if so.

## Phase 5 — Report

List every file touched, which vocabulary was applied (or "skipped — collides with existing lifecycle-state dots, left alone"), and the delivery outcome (committed / PR link / pending user confirmation).

_Dr. Mārcis Gasūns_
