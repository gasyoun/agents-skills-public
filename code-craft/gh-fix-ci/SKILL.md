---
name: "gh-fix-ci"
description: 'Debug + fix failing GitHub Actions checks — open-PR mode (default) or default-branch CI/Pages red repair, optional multi-repo fan-out. External providers (Buildkite) out of scope. Triggers: why is CI red, fix failing checks on this PR.'

---

# gh-fix-ci

Use this skill when the user asks to run the migrated source command `gh-fix-ci`.

## Command Template


# /gh-fix-ci [pr | --default-branch [repo…]]

_Created: 11-07-2026 · Last updated: 28-08-2026_

Locate failing GitHub Actions checks, fetch logs, summarize, plan, and implement. **Two modes** (H1661–H1778 skill-mine **C26** vote 28-07-2026 — UPDATE, not a separate `/red-branch-repair`):

| Mode | When | Scope |
|---|---|---|
| **open-PR** (default) | current branch PR, or `N` / PR URL | one PR; implement after approval |
| **default-branch** | `--default-branch` or "red on main/master" / "Pages red" | default branch health; optional multi-repo fan-out |

GitHub Actions only; external providers → URL only.

**Prereq:** `gh auth status` succeeds (repo + workflow scopes).

## Inputs

- **repo** — path inside the repo, default `.`.
- **pr** — PR number or URL; open-PR mode; defaults to the current branch's PR.
- **`--default-branch`** — switch to default-branch mode; optional trailing repo paths or slugs under `GitHub/`.
- **gh auth** — valid GitHub CLI auth (repo + workflow scopes).

## Quick start (open-PR)

```sh
python ~/.codex/skills/gh-fix-ci/scripts/inspect_pr_checks.py --repo "." --pr "<number-or-url>"
python ~/.codex/skills/gh-fix-ci/scripts/inspect_pr_checks.py --repo "." --pr "<url>" --json
python ~/.codex/skills/gh-fix-ci/scripts/inspect_pr_checks.py --repo "." --max-lines 200 --context 40
```

`--json` for machine-friendly output; exits non-zero while failures remain.

---

## Mode A — open-PR (default)

1. **Auth** — `gh auth status`. If unauthenticated, ask for `gh auth login`.
2. **Resolve the PR** — `gh pr view --json number,url` or the user-supplied number/URL.
3. **Inspect failing checks (GitHub Actions only)** — prefer the Quick-start helper. Manual fallback:
   ```sh
   gh pr checks <pr> --json name,state,bucket,link,startedAt,completedAt,workflow
   ```
   For each failing check, extract run id from `detailsUrl`:
   ```sh
   gh run view <run_id> --json name,workflowName,conclusion,status,url,event,headBranch,headSha
   gh run view <run_id> --log
   ```
   If still in progress: `gh api "/repos/<owner>/<repo>/actions/jobs/<job_id>/logs" > <path>`
4. **Scope out external checks** — non-Actions `detailsUrl` → report URL only.
5. **Summarize** — check name, run URL, concise log snippet; call out missing logs.
6. **Plan** — draft a fix plan; request approval before editing.
7. **Implement after approval** — apply, summarize diffs/tests, PR open/update as needed.
8. **Recheck** — re-run tests and `gh pr checks` until green.

---

## Mode B — default-branch CI/Pages red (C26 UPDATE)

Encodes the H1723 + H1736–H1743 + H1752 wave: fix **default-branch** red without inventing `/red-branch-repair`.

### B0 — Inventory

For each target repo (one, or a fan-out list):

```sh
gh api repos/<owner>/<repo> --jq ".default_branch"
gh run list --branch <default> --limit 15 --json conclusion,name,workflowName,url,headSha,createdAt,event
gh run list --branch <default> --status failure --limit 10
```

Also note GitHub Pages: Actions-based deploy vs legacy `pages-build-deployment` (may be structurally dormant after migrate to Actions — do not thrash a dead workflow; document and `@DECIDE` if needed).

### B1 — Classify the failure class (playbook)

Map the log to **one** class before coding:

| Class | Typical signal | Minimal fix pattern (evidence) |
|---|---|---|
| **Jekyll / Liquid** | `Liquid syntax error`, raw tags | wrap raw HTML/JSON in `{% raw %}` / escape (H1736, H1737) |
| **lychee / link audit** | private Uprava blob 404s in public repo | exclude private hosts, or rewrite to public targets (H1738) |
| **npm audit / dependency security** | `npm audit` fail on Pages/CI | pin/bump within policy, or scoped ignore with justification (H1739, H1740) |
| **CodeQL action pin mismatch** | init/autobuild/analyze different SHAs | pin **same** `codeql-action` SHA for all three steps (H1742) |
| **Pages deploy timeout** | Actions deploy-pages timeout | budget/path trim, artifact size, concurrency (H1741) |
| **Docker registry flake** | intermittent pull failures | retry/backoff, pin registry mirror (H1743) |
| **Dangling index / doc gate** | FINDINGS/Index false positive on headings | fix heading regex or source headings (H1752) |
| **Other** | novel | treat as open-PR Mode A on a fix branch off default |

### B2 — One repo, one minimal PR

- Branch off `origin/<default>` in a worktree when the repo is main-tree-guarded.
- Smallest fix that turns that class green; no drive-by refactors.
- Changelog `[Unreleased]` when the repo uses one; recheck with `gh run list` / latest workflow run on the PR and after merge on default.

### B3 — Optional multi-repo fan-out

When the user names a **red list** (or "fix all red default branches"):

1. Inventory all listed repos (B0).
2. Group by failure class (B1).
3. Fix **one class across repos** or **one repo fully** per session — do not intermix unrelated classes in one PR.
4. Report a table: repo · class · PR URL · green/still-red.

### Hard rules (Mode B)

- Prefer UPDATE of this skill over a second skill named `/red-branch-repair` (MG vote 28-07-2026).
- Do not force-push default branch; PR + merge only.
- Structurally dormant workflows → document, do not infinite-retry.
- External CI remains URL-only.

**This file is the canonical source.** The Codex + Grok twins
(`~/.codex/skills/gh-fix-ci/SKILL.md`, `~/.grok/skills/gh-fix-ci/SKILL.md`)
are GENERATED from it — never hand-edit them:
`python ~/.claude/scripts\generate_playbook_twins.py gh-fix-ci`.

_Dr. Mārcis Gasūns_
