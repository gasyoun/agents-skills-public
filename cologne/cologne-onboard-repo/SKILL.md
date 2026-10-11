---
name: "cologne-onboard-repo"
description: Bootstrap ONE new (or newly-adopted) repo with the full org baseline — CLAUDE.md/AGENTS.md, CODE_OF_CONDUCT, dependabot, CodeQL, pre-commit, branch protection, hub registration. Use when a repo was just created/forked (e.g. onboard kosha). Do not use for just writing a CLAUDE.md on an existing repo — that is /cologne-claude-md-fill; do not use to apply ONE deploy across many already-onboarded repos — that is /cologne-deploy-all.

---

# cologne-onboard-repo

Use this skill when the user asks to run the migrated source command `cologne-onboard-repo`.

## Command Template


# /cologne-onboard-repo

_Created: 03-07-2026 · Last updated: 05-09-2026_

## Default path (seconds — run this first)

```sh
python ~/Documents/GitHub/Uprava/tools/cologne_batch_deploy.py --repo <name> --dry-run
```

LLM / judgment residue only after the script report exists (or the script exits non-zero with a real defect). `cologne-claude-md-fill` stays after that report.

One new repo currently means hand-running the logic of five `*-all` batch skills plus doc seeding. This runbook composes them for a single target. **Do not reimplement any step — open the named runbook file and apply its per-repo core to this one repo.**

Argument: repo name (with owner if ambiguous — check `git remote -v`; gasyoun personal vs sanskrit-lexicon org matters for every URL emitted).

## Phase 0 — Identify

Confirm owner/org, default branch, language mix (`gh api repos/OWNER/REPO/languages`), and whether it is a dictionary repo, tooling repo, or personal project — this gates which steps apply.

## Phase 1 — Docs seed

- `CLAUDE.md`: apply `cologne-claude-md-fill.md` (investigate → write repo-specific guidance). Mirror the essentials into `AGENTS.md` for Codex.
- `.ai_state.md`: seed the standard 5-section skeleton (Objective / Next Steps / WIP / Dev Notes / Completed).
- `README`: only if missing/empty — short, with full blob URLs.

## Phase 2 — Governance + supply chain (one command, not seven skills)

[`Uprava/tools/cologne_batch_deploy.py`](https://github.com/gasyoun/Uprava/blob/main/tools/cologne_batch_deploy.py)
already consolidates the seven per-topic deploys (CODE_OF_CONDUCT, issue-templates,
dependabot, dependabot-automerge, codeql, pre-commit, branch-protect, gitattributes —
each a row in its `DEPLOYS` table) into one config-driven runner. Scope it to the one
new repo instead of hand-running seven separate `*-all` runbooks:

```powershell
python Uprava/tools/cologne_batch_deploy.py --repo <name> --dry-run   # preview: present/missing/upgrade per deploy
python Uprava/tools/cologne_batch_deploy.py --repo <name> --apply     # opens one PR per missing/stale deploy (gh_api rows apply directly)
```

Per-row `applies_if` already handles the skip conditions that used to be spelled
out per skill (already-present, no dependabot, PHP-only for CodeQL, no admin for
branch-protect surfaces as an `ERROR` line to route to GTD as `@DO`). Do not
re-derive or re-list those conditions here — read the row in `DEPLOYS` if in doubt.

**Not covered by this tool — still a separate step:**

| Step | Runbook | Skip when |
|---|---|---|
| SEO Pages landing page + .nojekyll | `cologne-pages-landing.md` | not a dictionary repo, or index.html is a real app/site |

## Phase 3 — Hub registration

- `Uprava/PROJECT_INTERLINKS.md`: add the repo if it produces or consumes a data feed.
- Issue taxonomy: if the repo will carry issues, run `cologne-issue-runbook.md` (dict) or `cologne-tooling-runbook.md` (tooling) — or record it as a queued step in `.ai_state.md`.
- `Uprava/GTD_NEXT_ACTIONS.md`: add any human-only leftovers (credentials, protection without admin, visibility decision).

## Phase 4 — Deliver

Baseline files go in as one PR to the new repo (auto-merge) unless the repo is minutes old and empty — then direct commit to default is acceptable. Hub edits commit in their own repos. Report a checklist of applied/skipped-with-reason per step.

_Dr. Mārcis Gasūns_
