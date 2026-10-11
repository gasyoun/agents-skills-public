---
name: "ai-state-journal-maintainer"
description: 'Maintain a repo''s .ai_state.md journal — canonical sections, move items between Queue/WIP/Notes/Completed instead of duplicating, concrete resume steps. Journal-only, no minting/GTD mirroring. Triggers: tidy .ai_state, cleanup before handoff.'

---

# ai-state-journal-maintainer

Use this skill when the user asks to run the migrated source command `ai-state-journal-maintainer`.

## Command Template


# /ai-state-journal-maintainer

_Created: 04-07-2026 · Last updated: 26-09-2026 (H5496: mechanical route ported)_

## Default path (seconds — run this first)

```sh
python3 ~/Documents/GitHub/Uprava/tools/ai_state_journal.py --file ./.ai_state.md                     # structural report (read-only dry-run)
python3 ~/Documents/GitHub/Uprava/tools/ai_state_journal.py --file ./.ai_state.md --normalize --rotate --max 10   # plan; add --apply to write
python3 ~/Documents/GitHub/Uprava/tools/ai_state_journal.py --file ./.ai_state.md --append queue --text "verify counts" --apply
```

The script parses the canonical sections below (exact heading text, tolerant keyword fallback for renamed headings), reports missing/duplicate sections and items outside sections, and performs the mechanical mutations — normalize (merge duplicates, park outside-section items in a labelled ❓ Unfiled section), append, rotate the Completed cap into a ✅ Completed Archive section — planning first, writing only with `--apply`. `--selftest` checks it offline.

Treat `.ai_state.md` as the **shared session journal**, not a dumping ground: current, concise, structured so the next agent resumes without re-discovering finished work. (For a full session-end ritual — handoff files, GTD mirroring, doc hygiene — use [`/handoff`](https://github.com/gasyoun/claude-config/blob/main/commands/handoff.md) instead; this command only grooms the journal.)

LLM / judgment residue only after the script report exists — the script is the structural pass; item semantics (what is actually done, what is genuinely next) come after it, by hand.

## Required structure (preserve the repo's exact heading text, incl. emoji)

```markdown
# Project Objective: [Global Goal]
## ➡️ Next Steps (Queue)
## 🚧 Current Work-In-Progress (WIP)
## 🧠 Dev Notes & Hypotheses (Bugs, ideas, context)
## ✅ Completed (Recent only)
```

## Workflow

1. Inspect the task, the current `.ai_state.md`, recent `git status`/log, and any issue/PR context that motivated the update.
2. **Move** items between Queue → WIP → Completed instead of duplicating them. Keep WIP to genuinely active work.
3. Record persistent bugs / failed hypotheses / changed approach under Dev Notes.
4. Record validation **commands + outcomes** for completed work — especially anything a PR body or handoff will cite.
5. Keep Completed recent; summarize older detail once it stops helping the next session (don't erase useful recent context just to shorten the file).
6. On handoff/stop: move finished work to Completed, state blockers explicitly, leave concrete Next Steps with exact file paths, issue/PR numbers, and commands.

## Boundaries

- Do **not** invent progress, and do not imply source work that wasn't actually done.
- Do not modify unrelated source files just to make the journal prettier.
- If the journal conflicts with repo facts, report the conflict and choose the evidence-backed state.

## Stop conditions

Stop / ask when the project objective is unclear, `.ai_state.md` has conflicting active goals, or an update would imply source work not actually done.

## Gate that stays (LLM + human)

The script never decides what counts as the CURRENT Next-Step: whether an item is genuinely done vs merely drafted, and where an unfiled item belongs (Queue vs WIP vs Notes) — it parks those in the labelled ❓ Unfiled section rather than guessing. That reading stays with the session's LLM under the stop conditions above.

_Mirrors the Codex skill `ai-state-journal-maintainer` (kept in sync via [`/claude-codex-skill-porter`](https://github.com/gasyoun/claude-config/blob/main/commands/claude-codex-skill-porter.md))._

_Dr. Mārcis Gasūns_
