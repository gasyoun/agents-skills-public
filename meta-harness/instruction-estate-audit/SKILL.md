---
name: "instruction-estate-audit"
description: 'Run the full instruction-estate census and contradiction audit. Use only when explicitly invoked as /instruction-estate-audit.'
---

# instruction-estate-audit

Use this skill when the user asks to run the migrated source command `instruction-estate-audit`.

## Command Template


# /instruction-estate-audit [--home <checkout>] [--days N] [--no-hooks]

_Created: 28-08-2026 · Last updated: 13-09-2026_ (C40)

Encodes the audit two sessions re-derived from scratch — [H1271](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1271-Fable_claude-config_instruction-estate-economics-audit_18.07.26.md) (Fable 5 — v1, chars/4) and [H3536](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H3536-Fable_claude-config_instruction-estate-economics-audit-v2_25.08.26.md) (Fable 5 — v2, real tokenizer, Stop layer, 15-card sheet [instruction_estate_v2_15](https://gasyoun.github.io/vote/sheets/instruction_estate_v2_15.html), 15/15) — whose three measuring scripts were never committed. Census card C40 ([SKILL_CANDIDATES_CENSUS_H3536_H3555_26_08_2026.md](https://github.com/gasyoun/Uprava/blob/main/SKILL_CANDIDATES_CENSUS_H3536_H3555_26_08_2026.md)), MG chat vote 28-08-2026 (6/6). Built via [H3618](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H3618-Fable_claude-config_instruction-estate-audit-skill_28.08.26.md).

The principle: **the measuring pass is a script, the judging pass is the model, and nothing is edited** — every reform is a card on a review sheet plus one `@DECIDE` row. The audit that edits rules while it measures them cannot be re-run.

Triggers: "estate audit", "how much does every session pay in instructions", "are the rules contradicting each other", "which rules earn their tokens", a `rules/` file added or a CLAUDE.md that grew, or roughly quarterly.

## Phase 1 — Census (deterministic, no LLM)

```
python scripts/instruction_estate_census.py --hooks            # markdown; --json for the machine copy
python scripts/instruction_estate_census.py --selftest         # must print "selftest OK" before you trust a number
```

[`scripts/instruction_estate_census.py`](https://github.com/gasyoun/claude-config/blob/main/scripts/instruction_estate_census.py) counts with `tiktoken o200k_base` (±10 % vs Anthropic's tokenizer; relative magnitudes hold; chars/4 shown only for H1271 comparability) over five load classes:

1. **always** — global `CLAUDE.md`, org-root `CLAUDE.md`, `rules/*.md`, the `MEMORY.md` index; `rules/` share and boilerplate tokens (dated line · "Always-on…" · byline) called out.
2. **per-prompt** — `--hooks` runs every SessionStart / UserPromptSubmit python hook **twice with one `session_id`** and reports both runs; the second run is the per-turn cost. A fresh id per run over-counts every once-per-session hook — the H3536 D6/P8 retraction.
3. **per-Stop** — `hook-blocks.jsonl` in the window: blocks, sessions, blocks per session, mean reason tokens, per-hook ranking, **repeat-identical reasons** (D5 class).
4. **repo** — `<repo>/CLAUDE.md` for `--repo` (default Uprava · Systema-Sanscriticum · SanskritLexicography).
5. **on-demand** — `references/*.md` sized, never loaded.

Plus the skill-load census from `playbook-invocations.jsonl` (demotion targets: a rule that fires only inside a skill loaded N times is skill-scoped).

**Home resolution trap:** `~` is the profile sandbox (`<claude-tools>\profiles\<n>`) on a profiled harness; the script walks `CLAUDE_CONFIG_DIR` → `$HOME/.claude` → `%USERPROFILE%\.claude` → `C:\Users\<user>\.claude` and takes the first tree with `settings.json` + `hooks/` + `CLAUDE.md`. Pass `--home <checkout>` to census a claude-config branch instead of the live install — **and diff the two**: a live tree that disagrees with `origin/main` is a finding in itself (28-08-2026: the live tree carried 11 rules while the applied vote had left 4).

## Phase 2 — Does each standing rule earn its cost?

One row per always-loaded rule, verdict from evidence only:

- **EARNS** — a hook logs violations of it in the window (n from Phase 1 per-hook table); the prose is not what carries compliance, the hook is.
- **UNMEASURED** — no hook, no ledger; only the ruling that created it. Say so; do not guess a rate.
- **MISPLACED** — live rule in the wrong load class: repo-scoped (needs a student identity, a Systema host), skill-scoped (fires only on a credential ask), a reference table wearing a rule's header, or a decision record.

**Compliance with vs without hook:** the only measured analogue is the best-instrumented prose rule (⭐ Next footer: 110 blocks/week while loaded on every turn). Extrapolate to unhooked rules explicitly as PLAUSIBLE, never as a number. Multi-homed rules (rule + memory `feedback_*` twin + CLAUDE.md paragraph + reference) are listed with every home.

## Phase 3 — Contradiction probes C1–C5 (scenario, not grep)

For each pair of rules that touch the same act, write the **concrete agent scenario where following A violates B** (migration deploy vs "destructive — confirm"; durable-doc-under-never-chat-only vs mint trigger; letter-coded options vs a Pass example that uses them; "publish every HTML" vs personal-data sheets; public `[integrity]` issue vs staff names). Verdict per row from an independent read-only [adversarial-verifier](https://github.com/gasyoun/claude-config/blob/main/agents/adversarial-verifier.md) pass: CONFIRMED (textual, same file or forced) · PLAUSIBLE (definitional gap, resolution exists but is not cross-referenced) · REFUTED (struck, kept in the doc as a struck row). List the non-conflicts you checked and dropped. Five is the floor, not the ceiling.

## Phase 4 — Stop-layer cost

A block costs its reason tokens **plus a regenerated reply** (300–1,500 output tokens, bounded not measured unless transcripts are parsed). Report blocks per session, the top repeat-identical reasons, and whether [`stop_dispatch.py`](https://github.com/gasyoun/claude-config/blob/main/hooks/stop_dispatch.py) `_suppress_repeat_blocks` is catching them (a reason repeating >20× across sessions in 7 days is an unfixed condition being re-served, not a compliance signal).

## Phase 5 — Proposals → sheet + one @DECIDE

Write `AUDIT_INSTRUCTION_ESTATE_<tag>.md` (+ `.meta.md`) in claude-config: Method · inventory tables · §2 verdicts · §3 contradictions · §4 structural defects · §5 proposals with **per-card saving in tokens/session and risk**. Every proposal is a card via [`/review-sheet`](https://github.com/gasyoun/claude-config/blob/main/commands/review-sheet.md), published to the vote hub; one `MG @DECIDE` row in [GTD_NEXT_ACTIONS.md](https://github.com/gasyoun/Uprava/blob/main/GTD_NEXT_ACTIONS.md) with the sheet URL, card count and 🟢/🟡/🔴. Apply nothing until the vote lands; the apply is its own handoff with a re-measure row (`--home` on the applied branch) in the decisions-applied record.

## Dedupe fence

- [`/hook-wiring-audit`](https://github.com/gasyoun/claude-config/blob/main/commands/hook-wiring-audit.md) — is a hook wired and firing; this skill asks what the firing **costs** and whether the prose it backs is needed.
- [`/claude-md-slim`](https://github.com/gasyoun/claude-config/blob/main/commands/claude-md-slim.md) — edits one CLAUDE.md; this skill never edits, it measures the whole estate and votes.
- [`/skill-effectiveness`](https://github.com/gasyoun/claude-config/blob/main/commands/skill-effectiveness.md) — skill loads over time; here they are only a demotion signal.
- [`/tier-audit`](https://github.com/gasyoun/claude-config/blob/main/commands/tier-audit.md) — model tiers, not instruction tokens.

Fail = a number from chars/4 or model recall · an edit to a rule inside the audit pass · a contradiction row without a scenario · a sheet URL missing from the @DECIDE row.

_Dr. Mārcis Gasūns_
