---
name: "cologne-deploy-all"
description: 'Run any of the seven mechanical org-wide deploys (branch-protect · codeql · dependabot · pre-commit etc.) via one deterministic script, org-wide or scoped. Mechanical batch, no LLM fan-out. Triggers: deploy X to all repos, add dependabot everywhere. Do not use to bootstrap a single newly-created repo — that is /cologne-onboard-repo; this applies one mechanical deploy across many already-existing repos.'

---

# cologne-deploy-all

Use this skill when the user asks to run the migrated source command `cologne-deploy-all`.

## Command Template


# /cologne-deploy-all &lt;deploy&gt; [--repo NAME]


## Default path (seconds — run this first)

```sh
python ~/Documents/GitHub/Uprava/tools/cologne_batch_deploy.py --dry-run --deploy <task>
```

LLM / judgment residue only after the script report exists (or the script exits non-zero with a real defect).

_Created: 20-07-2026 · Last updated: 05-09-2026_

Seven mechanical deploys used to be seven near-identical skill files that differed only in
the token they passed to the same script — consolidated into one config-driven script by
[`H556`](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H556-Sonnet_Uprava_cologne-batch-deploy-consolidation_11.07.26.md)
(Sonnet 5 — Uprava cologne-batch-deploy consolidation, 11-07-2026), and now into one skill. Every deploy scans the org under
`~/Documents/GitHub\` dynamically — no hardcoded repo list (the old static list
silently missed repos, e.g. SanskritLexicography, which has `requirements.txt` but was never
in `ALL_REPOS`).

```sh
# Read-only set-diff — no writes. ALWAYS run this first.
python ~/Documents/GitHub/Uprava/tools/cologne_batch_deploy.py --dry-run --deploy <deploy>

# Open one PR per repo currently missing the artifact (idempotent re-run)
python ~/Documents/GitHub/Uprava/tools/cologne_batch_deploy.py --apply --deploy <deploy>
```

Add `--repo <name>` to scope either form to a single repo.

## The seven deploys

| `--deploy` | What it lands | What to know |
|---|---|---|
| `branch-protect` | Branch protection on the default branch (`allow_force_pushes: false`, `allow_deletions: false`, `enforce_admins: false`, no required reviews — solo maintainer) | **Needs admin.** GitHub's API requires Pro (or a public repo) for private repos on free plans — `Upgrade to GitHub Pro…` on `--apply` is a plan limit, not a bug. For required CI checks, extend `BRANCH_PROTECTION_BODY` per repo (context names via `gh run view --repo {owner}/{repo} --json jobs --jq '.jobs[].name'`) — deliberately not automated, the choice is per-repo judgment |
| `code-of-conduct` | `CODE_OF_CONDUCT.md` (Contributor Covenant 2.1) | Source `coc_templates/CODE_OF_CONDUCT.md`; repos already having the file are skipped. Changing the contact email = edit the template once and re-run; replacing a *customized* existing file is a separate human-reviewed operation |
| `codeql` | `.github/workflows/codeql.yml` on repos with Python or JS/TS signals | ⚠️ **PHP is not a CodeQL language** — PHP SAST is Semgrep's job (see `csl-websanlexicon`'s `semgrep.yml`). Uses `codeql-action/init@v4` + `analyze@v4`, weekly Mon 03:00 UTC + push/PR, `build-mode: none`. Findings: GitHub → Security → Code scanning alerts |
| `dependabot` | `.github/dependabot.yml`, ecosystems auto-detected (`github-actions` always; `pip`/`npm`/`composer` when present) | Weekly, Mon 08:00 Europe/Moscow, PR limit 5. Only targets repos where the file is absent, so BookIndex's hand-crafted grouped-dependency config is never touched |
| `dependabot-automerge` | `.github/workflows/dependabot-auto-merge.yml` + flips on `allow_auto_merge` / `allow_squash_merge` / `delete_branch_on_merge` | Companion to `dependabot` (that opens the PRs, this merges them). Settings re-asserted on every `--apply` (idempotent). Needs `gh` auth with `repo`+`workflow` scope and admin for the settings PATCH. **Merges majors by default** — to hold them, gate the Approve+Enable steps on `steps.meta.outputs.update-type != 'version-update:semver-major'`. Pre-existing PRs don't retro-fire; clear a backlog once with `.sweep_dependabot_prs.py` |
| `issue-templates` | `bug_report.yml`, `text_correction.yml`, `question.yml`, `config.yml`, `PULL_REQUEST_TEMPLATE.md`, aligned with the org issue taxonomy | Source `issue_templates/`. **Skip rule:** a repo with ANY existing `.github/ISSUE_TEMPLATE/*.yml` — even a non-matching `feature_request.yml` — counts as customized and is left untouched (skip rather than risk clobbering) |
| `pre-commit` | `.pre-commit-config.yaml`, two tiers | **python+yaml** (repo has `.py`/`pyproject.toml`/`requirements*.txt`) gets `ruff-pre-commit v0.4.10` with `--select=E9,F63,F7,F8` (syntax errors only); **yaml-only** otherwise (`check-yaml`, `end-of-file-fixer`, `trailing-whitespace`, `check-merge-conflict`). Opt-in per contributor (`pip install pre-commit && pre-commit install`); CI already runs ruff via `lint-python` — this is local fast feedback |

Full config table: [`cologne_batch_deploy.py`](https://github.com/gasyoun/Uprava/blob/main/tools/cologne_batch_deploy.py).

## Not part of this skill

The judgment-gated `-all` siblings that stay separate: `/cologne-contributing-all`,
`/cologne-integrity-audit-all`, `/cologne-security-audit-all`. `/cologne-runbook-all`
and `/cologne-tooling-all` now call
[`cologne_issue_runbook.py`](https://github.com/gasyoun/Uprava/blob/main/tools/cologne_issue_runbook.py)
(the [`H2789`](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H2789-Sonnet_Uprava_zj-issue-runbook-detector_15.08.26.md)
(Sonnet 5) Uprava zj issue-runbook-detector port) — they are not `--deploy` tokens. Triaging a single Dependabot
PR's risk is [`/dependabot-pr-triage`](https://github.com/gasyoun/claude-config/blob/main/commands/dependabot-pr-triage.md).

`csl-orig` is never deployed to directly — agents don't commit there (see the org CLAUDE.md).

## Compatibility Rule

Write and follow this skill so it works in both Codex and Claude Code.

_Dr. Mārcis Gasūns_
