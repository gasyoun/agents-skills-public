# agents-skills-public — safe cut of the skills estate

_Public since 11-10-2026 · Owner: [gasyoun](https://github.com/gasyoun)_

A public, safety-vetted cut of selected agent skills (SKILL.md playbooks for AI coding harnesses:
Claude Code / OpenCode / Codex), carved out of the private umbrella monorepo
`agents-skills` (676 skills, private). **314 of 315** in-scope skills passed the audit and are
published here; 1 stays private (carried a personal email address).

## What's inside (11 groups)

| Group | Skills | What's inside |
|---|---|---|
| `meta-harness` | 66 | skill authoring/auditing, hooks, memory, docs, harness configs |
| `code-craft` | 48 | dev practice: TDD, review, debugging, CI, Playwright |
| `lexicography-pipeline` | 32 | lexicographic data pipeline: ingest, corrections, data releases |
| `web-ui-design` | 31 | UI/UX passes, motion, accessibility |
| `seo` | 31 | SEO family (audits, clusters, programmatic, backlinks) |
| `cologne` | 27 | Cologne/CDSL digital-lexicon ops: security triage, markup, issues |
| `analytics` | 23 | censuses, telemetry, metrics, quality scoring |
| `research-papers` | 19 | papers, venues, referee/review, submission packs |
| `sanskrit-linguistics` | 17 | Sanskrit: sandhi, grammar, parsing |
| `security-privacy` | 13 | security, privacy, publish gates |
| `pwg` | 7 | PWG_DE editorial/publishing pipeline |

Skills deploy flat: `~/.agents/skills/<name>/SKILL.md`.

## Safety audit (publish-safety-check)

Mechanical per-skill scan (committed scanner in the private umbrella: `tools/public_cut_safety_scan.py`)
against these exposure classes before publication:

- production infrastructure IPs (`.92`/`.91`) — **0 found** in scope
- personal/home filesystem paths — **9 skills redacted** (`/Users/...`, `C:\Users\...` → `<home>`, `<claude-tools>`)
- 152-FZ client identifiers — **7 skills redacted** (client repo name → `<client>`; 1 skill carried both classes)
- personal emails — **1 skill redacted** (public scholarly byline kept; personal Gmail holder excluded)
- secrets/tokens, student/staff names, phone numbers, money sums — **0 found** in scope

Post-redaction self-check: `--check` gate re-scans the published tree (skill dirs **and** root files) — **0 hits**.
Result: **GO**, 315 audited → 314 published (299 clean, 15 redacted), 1 excluded.

## What is NOT here

Business/student-data skills (education, payments, rosters), infra-ops (hosts, SSH, n8n),
org-handoffs/GTD, Russian-editing toolchain, slash-command mirrors — these live in the private
umbrella and are never published.
