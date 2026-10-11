---
name: "cologne-dict-triage"
description: Org-wide entry point for the SanskritSpellCheck HYBRID body-grounded dictionary triage (Sonnet 5 (claude-sonnet-5) classifies, Opus 4.8 (claude-opus-4-8) source-confirms, human verifies); runs one dictionary. Arg = MW, PW, PWG,… Do not use for GitHub issue triage — that is /cologne-bug-triage or /cologne-issue-runbook; this triages dictionary headwords via the SanskritSpellCheck HYBRID pipeline.
---

# cologne-dict-triage

Use this skill when the user asks to run the migrated source command `cologne-dict-triage`.

## Command Template


# Cologne dictionary triage — `$ARGUMENTS`  (global entry point)

_Created: 03-07-2026 · Last updated: 05-09-2026_

Org-wide wrapper for SanskritSpellCheck's body-grounded triage. The pipeline and its scripts
live **in the SanskritSpellCheck repo**; this skill routes there and is the source of truth's
front door, not a duplicate of it.

## 1. Locate and enter the repo
The pipeline must run from the **SanskritSpellCheck repo root** — its scripts are under
`detectors/`, output goes to `corrections_draft/`, and it reads the **`csl-orig`** sibling for
the dictionary source (`csl-orig/v02/<dict>/<dict>.txt`). The repo sits under the GitHub org
root, a sibling of `csl-orig` — typically `~/Documents/GitHub/SanskritSpellCheck`.
`cd` there. If it isn't present (or `csl-orig` isn't a sibling), say so and stop — ask the user
to clone `drdhaval2785/SanskritSpellCheck` next to `csl-orig`.

## 2. Follow the repo's canonical procedure
**Read `.claude/commands/dict-triage.md` in that repo and follow it verbatim for `$ARGUMENTS`.**
That file is the single source of truth for the steps — build the package
(`triage_dict.py $ARGUMENTS`) → launch `bodyaware_workflow.js` with the printed hybrid args →
`triage_dict.py $ARGUMENTS --finish` → human-verify each FILE-FIRST candidate against the entry →
write `corrections_draft/$ARGUMENTS/readme.md` → commit. Do **not** reconstruct the steps from
memory; read the repo skill so any later refinement there flows through automatically.

## Guardrails (hold even if you only skim)
- **Hybrid models**: Sonnet 5 (claude-sonnet-5) classifies the bulk, Opus 4.8
  (claude-opus-4-8) confirms the TYPO pile — pinned per-phase
  in `bodyaware_workflow.js`. No manual model toggling; it's one workflow run. (These pins
  predate Fable 5 — promoting the confirm phase to Fable 5 `claude-fable-5` means editing
  `bodyaware_workflow.js` in SanskritSpellCheck, never overriding the pin from this skill.)
- **The LLM TYPO pass is STOCHASTIC.** Re-runs surface a different small handful and can *lose*
  genuine typos — never blindly overwrite a committed, verified package with a fresh run. For
  recall, union across runs.
- **Source-verify every FILE-FIRST candidate yourself** (grep its `<k1>` entry): KEEP only when
  the entry's own derivation/citation confirms the suggestion (e.g. headword `arTavanDa` quoted
  as `lalitArTabanDaM`); DROP wrong-reading apparatus (`w.r.` / `fehlerhaft für` / `Richtig`),
  redirects (`{{Lbody}}` / `See` / `s.`), vṛddhi derivatives, and attested variants.
- **Tier-A precision is near-zero on mature dicts** (MW 4/1954 · PW 2/657 · VCP 1/563 · PWG
  12/497) — the standing **do-not-file list** + preventing bad bulk edits is the real value.
- For a dictionary whose body language isn't English/German/Sanskrit, add a `triage_lang.py`
  profile first (the repo skill's "New language" section).

This is a **skill, not an agent**: it's main-loop orchestration that launches a *nested Workflow*
(which spawns the Sonnet 5 / Opus 4.8 subagents), so it can't be a subagent itself.

_Dr. Mārcis Gasūns_
