---
name: "cologne-security-audit-all"
description: Org-wide security audit of Sanskrit Lexicon — GitHub Actions pwn-request/script-injection/token scope, committed secrets, SAST coverage. Read-mostly; PRs only for hardening. Sonnet 5 (claude-sonnet-5)/Opus 4.8 (claude-opus-4-8)/Fable 5 (claude-fable-5) tier. Do not use for one repo's CodeQL/Semgrep alerts (/cologne-alert-triage), PHP injection fixes (/cologne-php-xss-sweep), or data-integrity risks (/cologne-integrity-audit-all) — this is the org-wide Actions/secrets/SAST posture audit.

---

# cologne-security-audit-all

Use this skill when the user asks to run the migrated source command `cologne-security-audit-all`.

## Command Template


# /cologne-security-audit-all

_Created: 03-07-2026 · Last updated: 05-09-2026_

A periodic security pass over the whole org (`gasyoun/*` + `sanskrit-lexicon/*`). Covers the three cross-cutting surfaces that per-repo work misses: **CI workflow security**, **committed secrets**, and **SAST coverage**. Read-mostly — it ends in a ranked report; the only writes are small hardening PRs + alert dispositions you approve.

Run it monthly/quarterly, or after a big workflow-deploy session (the org-wide Dependabot/CodeQL/Semgrep rollouts make CI the highest-value thing to re-audit).

Argument `$ARGUMENTS` (optional): `nondict` (default — tooling/app repos), `dict`, or `all`.

---

## Conventions

- PR-only for fixes (branch off `origin/<default>`, never push default). Resolve the default branch per repo (not always `master`). Mind the **fork gotcha** (`--repo` on `gh pr create`).
- Windows: `sys.stdout.reconfigure(encoding='utf-8')`, `encoding='utf-8'` on `subprocess.run`, multi-step scripts to `.py`, no PowerShell script-blocks; write heavy scans to a `.py` and run in background.
- **Verify before dismissing.** A secret-scanner hit or SAST finding is a *candidate* — confirm the actual context before resolving it (see the Firebase/Cloud-key trap below). One mis-dismissal buries a real leak.

---

## Phase 0 — Enumerate + classify

`gh repo list <owner> --limit 200 --json name,primaryLanguage,isFork,isArchived,description`. Classify dict vs non-dict (non-dict = `csl-*`/`cologne-*` tooling + all `gasyoun/*` projects + frontends; dict = the lexica codes from the org CLAUDE.md). Triage surface from the **git tree** (count `.php/.py/.js/.mjs`, `.github/workflows/*`), not just the primary language — many "HTML" repos have PHP/Python backends. **Most repos now have workflows** (org-wide deploys), so CI is the universal surface.

## Phase 1 — GitHub Actions security (highest value)

For every workflow, flag and then **manually verify** the dangerous patterns:

- **Privileged triggers** — `pull_request_target`, `workflow_run`, `issue_comment`, `issues`. These run in the **base-repo context with secrets + a write token even for fork PRs**. A finding only if the workflow *also* checks out + executes PR-controlled code (**pwn-request**) or interpolates `${{ github.event.* }}` into a `run:`. Safe when: `permissions: contents: read`, gated to a trusted branch (`head_branch == 'main'`), and no PR-code execution.
- **Script injection** — `${{ github.event.* / github.head_ref / inputs.* / github.actor }}` interpolated directly into a `run:` shell block. **Exploitable** only if the trigger is reachable by untrusted input. NOT exploitable when the trigger is `workflow_dispatch`-only (requires repo-write → supplier is already trusted) — that's a *hardening* nit (move it to an `env:` var), fix via PR, don't alarm.
- **Not injectable**: a `type: choice` input (GitHub constrains it to the dropdown); an input placed in an `env:` block (value assignment, not shell); a controlled `html_url`/`PR_URL` used as a quoted arg.
- **Auto-merge** (`dependabot-auto-merge.yml`, ×~50, write perms + `gh pr merge`): **safe** when `on: pull_request` (not `pull_request_target` → fork PRs get a read-only token), `if: github.actor == 'dependabot[bot]'` (can't be spoofed), and **no PR-code checkout**. The documented direct-merge fallback on repos without required checks is a supply-chain tradeoff, not an injection bug.
- Token scope: prefer `GITHUB_TOKEN` + a `permissions:` block over a broad org PAT (`ORG_AUTOMERGE_TOKEN`-style) — flag broad PATs for least-privilege review.

## Phase 2 — Committed secrets

1. **GitHub secret-scanning alerts** (authoritative): `gh api repos/{slug}/secret-scanning/alerts?state=open`. Read each alert's `secret`, `first_location_detected`, and `validity`.
2. Tree scan for credential files (`.env`, `id_rsa`, `*.pem`, `credentials`, `.npmrc`) and a high-confidence inline grep (`sk-…`, `ghp_…`, `github_pat_…`, `AKIA…`, `AIza…`, `xox[bap]-…`, PEM headers) on **config-named files only** (config/settings/constants/.env) to stay bounded.

**Triage traps:**
- `.env.example` / `.env.*.example` = **placeholder templates** (empty values + setup instructions). By-design committed → false positive.
- **`AIza…` Google keys are NOT all the same.** A **Firebase web config** key (`apiKey` inside a `firebaseConfig{}` with `authDomain`/`projectId`/`storageBucket`) is **public by design** (Firebase Security Rules enforce access, not key secrecy) → dismiss as `wont_fix` with that rationale; recommend GCP key-restriction as defence-in-depth. A **Google Cloud API key** (e.g. a `GDRIVE = { apiKey, scopes: 'drive.file' }`, Maps, Translate) is a **real credential** → it needs **rotation in Google Cloud Console** (removal from current code does NOT un-leak it from public history). **Read the surrounding code before deciding** — the two look identical (`AIza…`) but the remediation is opposite. Keep the alert **open** until the user rotates.

## Phase 3 — SAST coverage + app code

- Confirm php-bearing repos route to **Semgrep** (CodeQL has no PHP analyzer — [[reference_codeql_no_php]]). App-code injection (reflected-XSS/SQLi) is handled by [`/cologne-php-xss-sweep`](https://github.com/gasyoun/claude-config/blob/main/commands/cologne-php-xss-sweep.md); note which repos are already swept vs pending. Most Python/JS is offline build tooling / static-site generation (no live untrusted-input endpoint) — low surface; spot-check any served endpoint (`$_GET`/`$_REQUEST`, a Flask/FastAPI route).

## Output

A ranked report: per surface, **confirmed** findings (with the exploit path) vs hardening nits vs false-positives-with-rationale. Then: open the hardening PRs, dispose of the SAST/secret alerts (dismiss FPs with written reasons; **reopen** anything mis-dismissed), and **surface** what only the user can do (rotate a leaked Cloud key, restrict a GCP key) — don't bury it.

See [[reference_codeql_no_php]], [[feedback_autonomous_task_prefs]]. Triage detail: [`/cologne-alert-triage`](https://github.com/gasyoun/claude-config/blob/main/commands/cologne-alert-triage.md).

## Agents (fan-out)

This skill pairs with the read-only agents in [`agents/`](https://github.com/gasyoun/claude-config/blob/main/agents/):

- **`cologne-security-reviewer`** — fan out one per code-bearing repo for the application-code surface.
- **`adversarial-verifier`** — verify every flagged workflow / secret finding before dismissing or escalating (it caught the Firebase-web-key-vs-Cloud-key distinction). Never bulk-dismiss without a verifier vote.

_Dr. Mārcis Gasūns_
