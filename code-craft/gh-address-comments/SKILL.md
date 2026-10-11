---
name: "gh-address-comments"
description: Find the open GitHub PR for the current branch and work through its review/issue comments with `gh` — list the threads, ask which to address, apply fixes for the chosen ones. Use for "address the PR comments", "handle review feedback on this branch".
---

# gh-address-comments

Use this skill when the user asks to run the migrated source command `gh-address-comments`.

## Command Template


# /gh-address-comments

_Created: 11-07-2026 · Last updated: 28-08-2026_

Find the open PR for the current branch and address its comments.

**Prereq:** `gh` is authenticated (`gh auth status`, repo + workflow scopes). If not, ask the user to run `gh auth login` (suggest `! gh auth login` to run it in-session); retry if `gh` hits an auth/rate issue mid-run.

## Workflow

1. **Inspect comments needing attention** — list every review thread and top-level comment on the current branch's PR. Portable path:
   ```sh
   gh pr view --json number,url
   gh api "/repos/<owner>/<repo>/pulls/<pr>/comments"          # review (inline) comments
   gh api "/repos/<owner>/<repo>/issues/<pr>/comments"          # issue-level comments
   gh api "/repos/<owner>/<repo>/pulls/<pr>/reviews"            # review summaries + states
   ```
   *Bundled helper (PC-only):* `python ~/.codex/skills/gh-address-comments/scripts/fetch_comments.py`.
2. **Number + summarize** — number all threads/comments; for each, give a one-line summary of what a fix would require.
3. **Ask which to address** — have the user pick the numbered comments (use `AskUserQuestion` if it helps).
4. **Apply fixes** for the selected comments only; summarize the diffs. Optionally reply to / resolve the threads via `gh api` once fixed.

**This file is the canonical source.** The Codex + Grok twins
(`~/.codex/skills/gh-address-comments/SKILL.md`, `~/.grok/skills/gh-address-comments/SKILL.md`)
are GENERATED from it — never hand-edit them:
`python ~/.claude/scripts\generate_playbook_twins.py gh-address-comments`.

_Dr. Mārcis Gasūns_
