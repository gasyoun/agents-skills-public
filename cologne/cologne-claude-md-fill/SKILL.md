---
name: "cologne-claude-md-fill"
description: Investigate a Sanskrit Lexicon satellite repo and write a repo-specific CLAUDE.md covering what it is, how to run, key files, CI, conventions. Use during cologne onboarding when a satellite repo lacks one. Judgment — Sonnet 5 (claude-sonnet-5) tier. Do not use for full repo bootstrap (dependabot, CodeQL, branch protection, hub registration) — that is /cologne-onboard-repo; this only writes the CLAUDE.md doc.

---

# cologne-claude-md-fill

Use this skill when the user asks to run the migrated source command `cologne-claude-md-fill`.

## Command Template


# /cologne-claude-md-fill

_Created: 03-07-2026 · Last updated: 05-09-2026_

Write a repo-specific `CLAUDE.md` for the Sanskrit Lexicon satellite repository given as the argument.

The org is `sanskrit-lexicon` (or personal fork under `gasyoun`). This skill investigates the repo from scratch and writes practical, accurate conventions — not a generic stub.

Execute **without per-repo confirmation**. Commit and push when done.

---

## Autonomy rules

- Execute all phases without user confirmation.
- `gh` API calls, file writes, git commits, and pushes are all pre-approved.
- Use `sys.stdout.reconfigure(encoding='utf-8')` in any Python scripts.
- Commit prefix: `ai-wip:`.

---

## Phase 0 — Setup

```
REPO=<argument>
REPO_PATH=~/Documents/GitHub/$REPO
```

Verify the repo exists locally:
```sh
ls $REPO_PATH/.git
```

Fetch remote metadata:
```sh
gh api repos/gasyoun/$REPO --jq '{name, description, language, topics}'
```

If 404, try `sanskrit-lexicon/$REPO`.

---

## Phase 1 — Skip if already substantive

Read `$REPO_PATH/CLAUDE.md` if it exists. Apply the same substantive markers as `/cologne-contributing-fix` Phase 1:
- ≥ 50 lines, AND contains repo-specific commands, paths, or architectural details.

If substantive: report "already substantive (N lines), skipping." and exit.

If missing or thin stub (< 20 lines of actual content): proceed to Phase 2.

---

## Phase 2 — Investigate the repo

Read enough to answer the six questions below. Use parallel reads where possible.

**Six questions:**

1. **What is it?** — One sentence: type (dictionary / tooling / content / ML / web app), purpose, audience.
2. **How do you run it?** — The exact commands to start dev, test, build, deploy. Look in: `package.json` scripts, `Makefile`, `pyproject.toml`, `README.md`, bat/sh launchers, `docker-compose.yml`.
3. **What are the key files and directories?** — Source, data, config, generated artifacts. Look at the top-level layout; read README for guidance.
4. **What CI workflows exist?** — Read `.github/workflows/*.yml`. Summarize trigger and purpose per workflow.
5. **What are the non-obvious conventions?** — Encoding requirements, branching model, generated-artifact policy, upstream fork relationship, special directories.
6. **What should NOT be touched?** — Generated files, vendored code, sensitive configs.

Sources to read (as needed):
- `README.md` — top 80 lines
- `package.json` — scripts section
- `pyproject.toml` / `setup.py` / `requirements*.txt`
- `.github/workflows/` — all workflow files
- `Makefile` / `Dockerfile` / `docker-compose.yml`
- Top-level directory listing

Do not read more than necessary. Stop when the six questions are answered.

---

## Phase 3 — Write CLAUDE.md

Write `$REPO_PATH/CLAUDE.md` using this structure:

```markdown
# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

<One paragraph: type, purpose, audience, any upstream relationship>

## Common commands

<The commands from question 2 — verbatim where possible>

## Key directories / files

<Table or bullet list from question 3>

## CI workflows

<Table: workflow file | trigger | purpose>

## Conventions

<Non-obvious rules from question 5>

## What not to touch

<List from question 6>
```

Rules:
- Use exact commands, not paraphrases. If a command was found in `package.json`, quote it exactly.
- Keep it short: aim for 50–120 lines. Dense and accurate beats long and generic.
- Use Russian for content descriptions if the repo's UI/docs are in Russian; use English for code identifiers and commands.
- Do not add sections for things you did not observe (e.g., omit "CI workflows" if none exist).
- Do not copy from a generic template — every section must be derived from what you actually read.

---

## Phase 4 — Commit and push

```sh
cd $REPO_PATH
git add CLAUDE.md
git commit -m "ai-wip: add CLAUDE.md with project conventions and commands

Per /cologne-claude-md-fill skill. Documents entry points,
key directories, CI, and non-obvious conventions.

Co-Authored-By: Claude <executing tier + version, e.g. Sonnet 5 (claude-sonnet-5)> <noreply@anthropic.com>"
git push
```

Update `.ai_state.md` under `## ✅ Completed` if the file exists:
```
- 2026-05-29: /cologne-claude-md-fill — wrote CLAUDE.md (N lines)
```

---

## Phase 5 — Final report

Print to the user:

```
Repo:     $REPO
Action:   created | updated | skipped (already substantive)
Lines:    N
Commit:   $SHA
```

---

## Notes for future use

- Run once per repo. For repos already having a substantive CLAUDE.md, this skill is a no-op.
- For batch operation across multiple repos, invoke sequentially (each repo needs its own investigation pass).
- The skill does not upgrade CLAUDE.md files created by earlier passes — if conventions change substantially, edit manually and commit.

_Dr. Mārcis Gasūns_
