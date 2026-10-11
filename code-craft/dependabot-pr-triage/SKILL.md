---
name: "dependabot-pr-triage"
description: 'Triage a Dependabot dependency-update PR (JS or Python) — classify patch/minor/major/security risk, read release notes, recommend merge/hold/close with evidence. Triggers: triage this Dependabot PR, is this bump safe to merge.'

---

# dependabot-pr-triage

Use this skill when the user asks to run the migrated source command `dependabot-pr-triage`.

## Command Template


# /dependabot-pr-triage

_Created: 11-07-2026 · Last updated: 29-08-2026_

## Default path (seconds — bulk sweep first)

```sh
python ~/Documents/GitHub/Uprava/tools/dependabot_pr_bulk.py
python ~/Documents/GitHub/Uprava/tools/dependabot_pr_bulk.py --owner gasyoun
python ~/Documents/GitHub/Uprava/tools/dependabot_pr_bulk.py --selftest
```

Script: [`Uprava/tools/dependabot_pr_bulk.py`](https://github.com/gasyoun/Uprava/blob/main/tools/dependabot_pr_bulk.py) (H1969 (Grok 4.5) — Uprava m2s leftovers/util cut-release W24–W25, which shipped the W2.4 bulk triager). Lists open Dependabot PRs with mechanical bump class + default outcome. **Does not merge.** Review `major-hold` / `security-urgent` / red CI with the residue below (or `dependency-bump-triager` per PR).

**Core rule:** dependency updates are *risk decisions*. Never merge or close on the version number alone — inspect ecosystem, update type, release notes, lockfile scope, and the repo's actual validation surface. Do not alter Dependabot branches unless the user explicitly asks for a rebase/recreate/local-fix/merge.

## Risk classes

- **patch-safe** — patch update, narrow dependency, normal release notes, lockfile scope as expected, validation passes.
- **minor-needs-checks** — minor update or tooling package; run build/tests before recommending merge.
- **major-hold** — major update / framework or runtime migration / peer-dependency shift; no merge without explicit migration review.
- **security-urgent** — security update; prioritize targeted validation and a merge path.
- **tooling-sensitive** — Playwright, Vite, Docusaurus, LangChain, model providers, test runners, bundlers, type systems; inspect regressions carefully.

## Per-PR delegation

**On a sweep of more than one open Dependabot PR, spawn the read-only
`dependency-bump-triager` subagent per PR, in parallel.** Each returns `RISK` × `OUTCOME`
+ `VALIDATION` + `EVIDENCE` against the two enums below; the release notes, changed-file
listings and an actual `npm test` / build run per PR are the dirty context this session
must not accumulate across a whole repo's queue. Merging, closing, holding and every
`gh` write stay here — the agent recommends only. A single-PR invocation can run the
checklist inline.

## Inspection checklist

1. Confirm the author is Dependabot; capture repo, PR#, package, ecosystem, old→new version, semver class.
2. Read the Dependabot body, labels, changed files, and release-note / changelog excerpts.
3. Inspect changed files: manifest-only, lockfile, generated files, or source changes.
4. Read release-note bullets for runtime / API / type / CI / loader / browser / Node-or-Python-version changes.
5. Note whether a compatibility score, labels, or CI status are present.

## Minimal validation

Use the repo's **actual** scripts from `package.json` — do not invent commands. For npm/Docusaurus repos:
```sh
npm test
npm run build
```
For Python repos, inspect `requirements*.txt` / `pyproject.toml` / project docs, then run the narrowest available test or import/build check. If a local checkout isn't available, state the checks that *should* gate merge.

## Outcome wording

Recommend exactly one, always with evidence (release-note signal, changed files, commands run, residual risk):
- **merge** — validation passed, risk low.
- **hold** — needs human review, broader CI, or an upstream-issue check.
- **close/ignore** — update unwanted, incompatible, or superseded.
- **needs-fix** — acceptable update but the local repo must change first.

## Stop conditions

Stop / ask when: the PR is not from Dependabot; release notes are unavailable for a risky update; validation can't run locally; the lockfile includes unexpected packages; or the package is build/test infrastructure likely to affect CI.

_Dr. Mārcis Gasūns_

_Frequent targets: `csl-guides`, `RuWritingStyles`, `Systema-Sanscriticum`, `SanskritKaraoke`. Mirrors the Codex skill `dependabot-pr-triage` (kept in sync via [`/claude-codex-skill-porter`](https://github.com/gasyoun/claude-config/blob/main/commands/claude-codex-skill-porter.md))._

_Dr. Mārcis Gasūns_
