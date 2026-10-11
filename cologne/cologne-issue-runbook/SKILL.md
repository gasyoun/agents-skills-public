---
name: "cologne-issue-runbook"
description: Apply the Sanskrit Lexicon issue-taxonomy runbook to one dictionary repo. Mechanical detector + label applicator is a script; type assignment stays LLM. Use to triage/label a single sanskrit-lexicon dictionary repo. Do not use org-wide — that is /cologne-runbook-all; do not use for a single already-labeled bug or question issue — that is /cologne-bug-triage or /cologne-question-research.
---

# cologne-issue-runbook

Use this skill when the user asks to run the migrated source command `cologne-issue-runbook`.

## Command Template


# /cologne-issue-runbook <REPO>

_Created: 03-07-2026 · Last updated: 05-09-2026_


## Default path (seconds — run this first)

```sh
python ~/Documents/GitHub/Uprava/tools/cologne_issue_runbook.py --repo <REPO>
```

LLM / judgment residue only after the script report exists (or the script exits non-zero with a real defect).

The org is always `sanskrit-lexicon`. The script is the detector, the label/milestone/project applicator, and the verification gate. It does **not** invent a type.

[`tools/cologne_issue_runbook.py`](https://github.com/gasyoun/Uprava/blob/main/tools/cologne_issue_runbook.py) (H2789, Sonnet 5 — ZJ issue-runbook detector). `--dry-run` is the default.

```sh
# Read-only: PROCESSED / UNPROCESSED / skipped / ERROR + planned writes
python ~/Documents/GitHub/Uprava/tools/cologne_issue_runbook.py --repo <REPO>

# After a plan.json exists (see Judgment below)
python ~/Documents/GitHub/Uprava/tools/cologne_issue_runbook.py --apply --repo <REPO> --classifications plan.json
```

A second `--apply` on an already-complete repo is a no-op. Never `--apply` before reading the dry-run.

Do **not** re-embed `<repo>_label.py` or a first-page REST `…/issues?per_page=100` fetch. Those are the defects this script replaced.

## Judgment (after the script report)

Only for issues the dry-run lists as `needs_judgment` (`no-type` or `multi-type`).

Every issue gets **exactly one** type. After the correct type is known, put it in `plan.json` and let the script apply default severity + milestone + project, and delete stale defaults named in `strip`.

```json
{
  "224": {"type": "question", "strip": ["bug"]}
}
```

`sev` and `ms` are optional — omitted values come from the type→default tables in the script.

| Label | When to apply |
|---|---|
| `link-target` | Building click-throughs from `<ls>` abbreviations to scanned PDF pages |
| `link-splitting` | Splitting combined `SOURCE N,N` refs into individual per-page links |
| `markup` | Normalising XML tag content (`<ls>`, `<lex>`, `<ab>`, etc.) |
| `text-correction` | Corrections to German/English definitions or Sanskrit headwords |
| `content-enhancement` | New material, display upgrades, structural additions beyond correction |
| `encoding` | SLP1/AS/IAST transcoding, character rendering, hyphen/dash normalisation |
| `scan-quality` | Replacing blurry, skewed, or missing scan pages |
| `bug` | Broken links, XML errors, broken download files |
| `question` | Scholarly questions requiring research before any code change |

Escalate severity to `hard` only for very large issues — write `"sev": "hard"` in the plan. Defaults: `link-target`/`link-splitting`/`content-enhancement` → `medium`; everything else → `minor`.

Keep non-type GitHub defaults (`invalid`, `duplicate`, `wontfix`, `help wanted`). After the correct type is assigned, `strip` any leftover default `bug`/`question` that is not that type.

Noise titles (`test`/`admin`/`note`/`meta`; zero labels + zero comments + open 5+ years) are [ASK] once. Do not invent a skip inside the script.

## Docs tail (still LLM)

Phases 8–16 of the old runbook (CLAUDE.md, README live counts + Mermaid, CITATION.cff, Source block, encoding, pipeline DAG, community files) stay here. Tooling README regen already lives in [csl-observatory `tooling_runbook.py refresh`](https://github.com/sanskrit-lexicon/csl-observatory/blob/main/scripts/tooling_runbook.py) — do not rebuild it. Community file copy is [csl-observatory `propagate_templates.py`](https://github.com/sanskrit-lexicon/csl-observatory/blob/main/scripts/propagate_templates.py).

Preserve `<!-- BEGIN MANUAL` / `## Documentation` / `doc/`/`docs/` links byte-for-byte when touching a README.

## Not this skill

- Org-wide dictionary sweep → `/cologne-runbook-all` (same script, `--taxonomy dictionary`).
- Tooling repos → `/cologne-tooling-runbook` (judgment) after `/cologne-tooling-all` (same script, `--taxonomy tooling`).
- `/cologne-tooling-runbook` itself is **not** folded (H556, Sonnet 5 — cologne-batch-deploy consolidation).

_Dr. Mārcis Gasūns_
