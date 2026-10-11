---
name: "cologne-runbook-all"
description: Discover every unprocessed sanskrit-lexicon dictionary repo and run the issue-taxonomy detector. Spawn LLM only for repos the script lists as UNPROCESSED with needs_judgment issues. Do not use for non-dictionary tooling repos — that is /cologne-tooling-all; do not use for a single already-identified repo — that is /cologne-issue-runbook.

---

# cologne-runbook-all

Use this skill when the user asks to run the migrated source command `cologne-runbook-all`.

## Command Template


# /cologne-runbook-all

_Created: 03-07-2026 · Last updated: 05-09-2026_


## Default path (seconds — run this first)

```sh
python ~/Documents/GitHub/Uprava/tools/cologne_issue_runbook.py --taxonomy dictionary
```

LLM / judgment residue only after the script report exists (or the script exits non-zero with a real defect).

[`tools/cologne_issue_runbook.py`](https://github.com/gasyoun/Uprava/blob/main/tools/cologne_issue_runbook.py) (H2789, Sonnet 5 - ZJ issue-runbook detector) discovers `sanskrit-lexicon` repos live, routes with the skip predicates from this skill (never a hardcoded "already processed" list), and prints `PROCESSED` / `UNPROCESSED` / `empty` / `skipped(reason)` / `ERROR` per repo.

A repo is processed only when **every** issue has exactly 1 type + 1 severity + 1 milestone + the matching org project. The 2026-05-31 false-done incident was a label-*definition* check — the script does not do that.

```sh
# Org-wide read-only set-diff (default)
python ~/Documents/GitHub/Uprava/tools/cologne_issue_runbook.py --taxonomy dictionary

# One repo
python ~/Documents/GitHub/Uprava/tools/cologne_issue_runbook.py --repo PWG
```

`--apply` only after a human has read the dry-run. It still will not invent a type.

## Judgment (after the script report)

For each repo printed `UNPROCESSED` whose remaining gaps are `needs_judgment`:

1. Read [`/cologne-issue-runbook`](https://github.com/gasyoun/claude-config/blob/main/commands/cologne-issue-runbook.md).
2. Classify only the `no-type` / `multi-type` issues.
3. Write `plan.json` and run `--apply --repo NAME --classifications plan.json`.
4. Docs tail (CLAUDE.md / README / citation) stays in that skill.

`empty` and `PROCESSED` are skipped. `skipped(meta|archive)` is the routing predicate, not a defect.

Do **not** paste the old inline discovery script into a chat. Do **not** spawn a sub-agent per repo before the script report exists.

## Notes

- GitHub primary rate limit is 5,000 req/hour. The script goes through `gh_ratelimit.gh_run`.
- `/cologne-runbook-loop` is a deprecated alias of this skill.
- `/cologne-tooling-runbook` is not folded.

_Dr. Mārcis Gasūns_
