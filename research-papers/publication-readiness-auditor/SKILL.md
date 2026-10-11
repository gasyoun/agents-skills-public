---
name: "publication-readiness-auditor"
description: 'Audit an article''s readiness across trackers (manuscript, cover letter, ARTICLES/GTD hubs, ORCID/byline, journal requirements) → ready/not-ready/blocked verdict. Read-only; pairs with /paper-referee + /paper-submission-pack. Trigger: "is A## still ready".'

---

# publication-readiness-auditor

Use this skill when the user asks to run the migrated source command `publication-readiness-auditor`.

## Command Template


# /publication-readiness-auditor

_Created: 04-07-2026 · Last updated: 26-08-2026_

**Audit before editing.** Publication readiness is a claim spread across manuscript, cover letter, readiness checklist, project hub, roadmap, and journal gates — never mark an item ready from one file. This command *audits and reports*; it does not push a paper to readiness (that's [`/paper-referee`](https://github.com/gasyoun/claude-config/blob/main/commands/paper-referee.md)) or package it (that's [`/paper-submission-pack`](https://github.com/gasyoun/claude-config/blob/main/commands/paper-submission-pack.md)).

## Evidence sources

- Manuscript frontmatter + body.
- Cover / submission letter.
- Readiness checklist / report.
- Hub files: `ARTICLES`, GTD, roadmap / publication plan.
- `.ai_state.md` recent completed work + open gates.
- Repo rules: `CLAUDE.md` / `AGENTS.md` / validation docs; canonical author source (often `Uprava/AUTHOR.md`).

## Audit checklist

- Readiness score + status wording agree across all trackers.
- Title, article ID, target journal, submission stage are consistent.
- Author name/byline form matches the journal's requirement (Cyrillic-vs-Latin is a gate, not an auto-fix).
- ORCID + email present where required and match the canonical source.
- Open gates are explicit, countable, and not silently closed.
- Bibliography, forbidden strings, and repo-specific validation rules pass.
- Generated artifacts are refreshed or listed as follow-up.

## Workflow

1. Identify the publication unit: article number, title, target journal, manuscript path, cover/readiness docs.
2. Inspect repo rules, `.ai_state.md`, and all trackers that mention the unit.
3. Cross-check readiness score, status text, author metadata, byline form, ORCID/email, open gates, sign-off state.
4. Separate factual fixes from author/editor **decisions** — do not close decision gates without evidence.
5. Run the repo's validation (e.g. `python scripts/validate.py`) or document why none applies.
6. Report a verdict with exact files + next actions.

## Verdicts

- **ready** — all required evidence consistent, validation passes.
- **not ready** — known gates remain and are documented.
- **inconsistent** — trackers disagree; list the exact conflicts.
- **blocked** — needs author / editor / archive / external decision.

## Stop conditions

Stop and report when target-journal requirements are unknown, author sign-off is needed, metadata sources conflict, or readiness depends on external archival/editorial work.

_The Codex/Grok twins of this playbook are GENERATED from this canonical file (`scripts/generate_playbook_twins.py`) — never hand-edit them; see [`/playbook-canonical-source`](https://github.com/gasyoun/claude-config/blob/main/commands/playbook-canonical-source.md)._

_Dr. Mārcis Gasūns_
