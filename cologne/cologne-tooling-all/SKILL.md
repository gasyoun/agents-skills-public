---
name: "cologne-tooling-all"
description: Discover every unprocessed non-dictionary sanskrit-lexicon repo via the shared issue-taxonomy script. Spawn /cologne-tooling-runbook only for UNPROCESSED tooling repos. Do not use for dictionary repos — that is /cologne-runbook-all; this discovers and fans out over non-dictionary tooling repos only.

---

# cologne-tooling-all

Use this skill when the user asks to run the migrated source command `cologne-tooling-all`.

## Command Template


# /cologne-tooling-all

_Created: 11-07-2026 · Last updated: 28-08-2026_


## Default path (seconds — run this first)

```sh
python ~/Documents/GitHub/Uprava/tools/cologne_issue_runbook.py --taxonomy tooling
```

LLM / judgment residue only after the script report exists (or the script exits non-zero with a real defect).

[`tools/cologne_issue_runbook.py`](https://github.com/gasyoun/Uprava/blob/main/tools/cologne_issue_runbook.py) (H2789, Sonnet 5 - ZJ issue-runbook detector) is the same detector as `/cologne-runbook-all`,with the tooling TYPE/SEV set and `discussion` milestone-exempt. Routing (dictionary / archive / meta) is a predicate, not a memorized processed list.

```sh
python ~/Documents/GitHub/Uprava/tools/cologne_issue_runbook.py --taxonomy tooling
python ~/Documents/GitHub/Uprava/tools/cologne_issue_runbook.py --repo csl-orig
```

A tooling repo is processed when every issue has exactly 1 type + 1 severity + a milestone (`discussion` is milestone-exempt). `COLOGNE` and `CORRECTIONS` print `skipped(meta)` (DECISIONS_NEEDED A2). Archives print `skipped(archive)`. Dictionary repos are not in this taxonomy — they belong to `/cologne-runbook-all`.

`--apply` only after a human has read the dry-run. The script never invents a type.

## Judgment (after the script report)

`/cologne-tooling-runbook` stays LLM-gated (H556, Sonnet 5 - cologne-batch-deploy consolidation) — category heuristic, type semantics, domain labels. Spawn it only for rows printed `UNPROCESSED` with `needs_judgment` issues:

```
File: ~/.claude/commands\cologne-tooling-runbook.md
ORG = sanskrit-lexicon
REPO = <name from the script report>
```

Do **not** spawn before the script report. Do **not** paste the old inline discovery Python.

`csl-observatory` may still carry leftover dictionary labels — the script will list them as `multi-type` / `no-type`; the tooling runbook strips the dictionary types before applying tool types.

## Not this skill

- Per-repo type assignment → `/cologne-tooling-runbook` (not folded).
- Dictionary sweep → `/cologne-runbook-all`.

_Dr. Mārcis Gasūns_
