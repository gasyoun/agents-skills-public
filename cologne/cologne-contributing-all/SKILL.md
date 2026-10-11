---
name: "cologne-contributing-all"
description: 'Apply /cologne-contributing-fix to every Sanskrit Lexicon repo in parallel — pre-classify substantive/boilerplate/missing, cap 4 concurrent, roll-up report. Sonnet 5 (claude-sonnet-5) tier. Triggers: "org-wide contributing standard", "CONTRIBUTING sweep". Do not use for a single repo — that is /cologne-contributing-fix; this fans it out org-wide.'

---

# cologne-contributing-all

Use this skill when the user asks to run the migrated source command `cologne-contributing-all`.

## Command Template


_Created: 03-07-2026 · Last updated: 26-09-2026 (H5490: mechanical route ported)_

## Default path (seconds — run this first)

```sh
python3 ~/Documents/GitHub/Uprava/tools/cologne_contributing_all.py --dry-run    # classify every org repo: missing / boilerplate / substantive / stub-present (offline, read-only)
python3 ~/Documents/GitHub/Uprava/tools/cologne_contributing_all.py --apply      # deploy the canonical stub to missing+boilerplate via branch+PR (network); substantive files never touched
python3 ~/Documents/GitHub/Uprava/tools/cologne_contributing_all.py --selftest
```

LLM / judgment residue only after the script report exists — the dry-run table IS Phase 2's classification roll-up, and `--apply` IS Phases 0/3 mechanically (via the proven `cologne_batch_deploy` engine: Contents-API PRs, idempotent, rate-limit-aware).

**The gate that stays: substantive-vs-boilerplate edge cases.** The classifier is deterministic (marker phrases, line count, repo-specific signals); everything it calls `substantive` is untouchable, and a borderline file — short but with content the markers miss, or a stub dressed as repo-specific — is escalated to the Phase 2 **[ASK]** keep-list ruling before `--apply` runs. `csl-orig` stays excluded on apply (org rule: no agent PR noise there — route via `/cologne-correction-queue`).

Apply [`/cologne-contributing-fix`](https://github.com/gasyoun/claude-config/blob/main/commands/cologne-contributing-fix.md) to **every** Sanskrit Lexicon repository under `~/Documents/GitHub\` in parallel.

The org is `sanskrit-lexicon`. This batch driver discovers all local repos, classifies each one (substantive / boilerplate / missing), runs the per-repo skill, and emits a single roll-up report.

Execute **without per-repo confirmation**. Ask once at Phase 2 [ASK] to confirm the substantive-keep list before any writes.

---

## Phase 0 — Bootstrap the org-wide standard first

Before touching any other repo, run `/cologne-contributing-fix COLOGNE`. The COLOGNE repo holds the canonical standard that every other stub links to — it must exist and be correct before stubs are written elsewhere.

Verify the standard is in place:
```sh
curl -sf https://raw.githubusercontent.com/sanskrit-lexicon/COLOGNE/main/CONTRIBUTING.md >/dev/null
```
If the curl fails, the COLOGNE push hasn't propagated yet — wait 30s and re-check before continuing.

---

## Phase 1 — Discover repos

```sh
ls -d <home>/Documents/GitHub/*/ | xargs -n1 basename
```

Filter to repos that:
- Have a `.git` directory (skip `files/`, `scripts/` and similar non-repos)
- Have `remote.origin.url` matching `sanskrit-lexicon/*` OR are gasyoun's CDSL satellites (e.g., `BookIndex`, `Systema-Sanscriticum`, `ORS-FAQ`) — judgment call by remote URL

Write the list to `/tmp/cologne_contrib_targets.txt`.

---

## Phase 2 — Pre-classify all repos (read-only)

Run a fast Python sweep that classifies each target without writing anything. Write the script to `classify_contrib.py` (UTF-8, with `sys.stdout.reconfigure(encoding='utf-8')`).

For each repo, decide `substantive | boilerplate | missing` by the same rules as `/cologne-contributing-fix` Phase 1. Output a table:

```
Repo                  Lines  Verdict       Reason
--------------------- ------ ------------- ----------------------------------
MWS                   120    substantive   has repo-specific PR workflow + setup
csl-observatory       110    substantive   has roles, authorship policy
PWG                   6      boilerplate   generic fork/branch/PR only
COLOGNE               17     boilerplate   restates taxonomy verbatim
...
```

**[ASK once]** Present the substantive list to the user. Confirm none of them should be downgraded to a stub. Default to keep on no response.

---

## Phase 3 — Parallel execution

For each repo in the target list, apply the `/cologne-contributing-fix` procedure per repo — fan out via the Agent tool (one sub-agent per repo), capping ~4 concurrent to avoid `gh api` rate limits. Slash commands are harness-level (invoked through the Skill/Agent layer), **not** shell executables — do NOT try to run `/cologne-contributing-fix` inside a `Bash` job; that silently does nothing.

Track progress in `/tmp/cologne_contrib_log.txt`:
```
[12:01:03] START   PWG
[12:01:08] DONE    PWG    verdict=boilerplate  commit=abc1234
[12:01:09] START   AP90
...
[12:05:42] DONE    csl-orig  verdict=missing  commit=def5678
```

Monitor until all `START` lines have matching `DONE` or `ERROR` lines. On error, capture stderr to `/tmp/cologne_contrib_errors/$REPO.log` and continue with the rest — do not abort the batch.

---

## Phase 4 — Roll-up report

Once all repos finish, emit a single markdown table to `<home>/Documents/GitHub/CONTRIBUTING_AUDIT.md`:

```markdown
# CONTRIBUTING.md Standardisation — Roll-up Report

**Run date:** 2026-05-28
**Repos processed:** N
**Substantive (kept):** A
**Boilerplate (replaced):** B
**Missing (created):** C
**Errors:** D

## Per-repo outcomes

| Repo | Verdict | Lines before → after | Commit |
|---|---|---|---|
| MWS | substantive | 120 → 121 | abc1234 |
| PWG | boilerplate | 6 → 24 | def5678 |
| ... | ... | ... | ... |

## Errors

| Repo | Stage | Message |
|---|---|---|
| ... | ... | ... |

## What changed in the substantive repos

For each kept repo, list only the diff hunks (typically the one-line inherit pointer added at the top).
```

---

## Phase 5 — Commit the audit report

The audit report lives in the parent `GitHub/` directory (not in any individual repo). Do not commit it — it's a working artifact. Mention its path in the final response.

---

## Phase 6 — Final response to user

Print to the user:

```
✅ CONTRIBUTING.md standardisation complete.

Substantive (kept):   A repos
Boilerplate (rewrote): B repos
Missing (created):    C repos
Errors:               D repos

Report: ~/Documents/GitHub\CONTRIBUTING_AUDIT.md
```

If any errors, list the affected repos under "Needs manual review".

_Dr. Mārcis Gasūns_
