---
name: "cologne-markup-batch"
description: 'Cologne dictionary markup-normalization batch (ls/ab/lex/h/info tags, abbreviation/link markup) across CDSL repos once meaning-dependent interpretation is resolved; batch of /cologne-markup-fix. Triggers: "normalize the markup", "batch-fix these tags". Do not use for a single dictionary''s markup-fixer and audit — that is /cologne-markup-fix; this is the CDSL-wide batch, run once meaning-dependent interpretation is already resolved.'

---

# cologne-markup-batch

Use this skill when the user asks to run the migrated source command `cologne-markup-batch`.

## Command Template


# /cologne-markup-batch

_Created: 04-07-2026 · Last updated: 05-09-2026_

Markup batches are correction work with a **wider blast radius** than a single fix. Prefer small, inspectable batches; preserve dictionary text; validate generated XML / ET parsing before calling the batch done. One issue, one dictionary, one markup family per batch.

## Preflight

- `git status --short --branch`.
- Read `.ai_state.md` if present; keep its required sections intact.
- Markup issues belong under the **Structured Data** milestone (see the org issue taxonomy).
- Identify the canonical source workflow before editing:
  - XML repos use generated source XML + local validation scripts.
  - `csl-orig` corrections edit `csl-orig/v02/{dict}/{dict}.txt` and require an audit change file in `csl-corrections`.
  - Many repos use `python updateByLine.py <input> <changefile> <output>`.

## Classify first — bail out if not markup-only

Classify the requested change as markup / text-correction / encoding / bug / editorial-question. Switch skills when it is not markup-only:
- wording/headword/definition corrections → [`/cologne-text-correction-pr`](https://github.com/gasyoun/claude-config/blob/main/commands/cologne-text-correction-pr.md).
- scholarly or editorial uncertainty → [`/cologne-question-research`](https://github.com/gasyoun/claude-config/blob/main/commands/cologne-question-research.md).

**Markup-only preflight:** continue only when the meaning of the tag/abbreviation/marker/lexical convention is already clear. If a change to `<ab>`, `<h>`, `<lex>`, `<info>`, or a similar marker depends on interpreting dictionary meaning or editorial policy, run `/cologne-question-research` first.

## Pattern discovery

- Use `rg` for exact bad patterns, tag names, abbreviations, and nearby record IDs.
- Sample enough matches to prove the replacement rule is not overbroad; check at least one **negative** example that must NOT change.
- For a polysemous pattern, prove positive AND negative examples semantically, not just syntactically (e.g. is `s. v.` one unit? does `<h>` carry the intended local meaning here?).
- Preserve line order and UTF-8 **without BOM**.

## Change construction

- Prefer a change file with paired old/new lines when the repo supports it.
- Use a short checked-in script only when the batch is too large/fragile for a hand-authored change file.
- Keep one markup family per batch (`<ls>` normalization, `<ab>` placement, `<lex>` cleanup, link splitting). Do not combine markup cleanup with translation/headword/definition changes.
- Do not silently edit scan-faithful files such as `printchange.txt` unless the task is explicitly about print-deviation notes.

## Validation

- Run the smallest focused check first (parser, unit test, exact-pattern count).
- For `csl-orig`, generate through `csl-pywork/v02/generate_dict.sh {dict} tempparent/{dict}` when available. On local Windows without XAMPP, `make_xml.py` reporting `All records parsed by ET` is the accepted XML-validation signal.
- Verify no BOM: first three bytes must not be `efbbbf`.
- Record the exact validation command + result in `.ai_state.md`.

## Handoff

When stopping or handing off, record: issue/task ID · dictionary/repo + source files · exact pattern changed · files generated or intentionally not generated · validation command + outcome · remaining candidates or blockers. Micro-commit logical milestones with `ai-wip:` when git access is available.

_Mirrors the Codex skill `cologne-markup-batch` (kept in sync via [`/claude-codex-skill-porter`](https://github.com/gasyoun/claude-config/blob/main/commands/claude-codex-skill-porter.md))._

_Dr. Mārcis Gasūns_
