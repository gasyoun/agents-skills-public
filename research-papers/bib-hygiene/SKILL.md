---
name: "bib-hygiene"
description: 'Validate + dedup .bib files — duplicate cite-keys, same-DOI/title-clash entries, missing fields, malformed syntax — fix-list + safe fixes. Sonnet 5 (claude-sonnet-5), Haiku 4.5 (claude-haiku-4-5) fallback. Triggers: "bib hygiene", "validate the .bib".'
---

# bib-hygiene

Use this skill when the user asks to run the migrated source command `bib-hygiene`.

## Command Template


# /bib-hygiene

_Created: 14-07-2026 · Last updated: 28-08-2026_

## Default path (seconds — run this first)

```sh
python ~/Documents/GitHub/Uprava/tools/bib_hygiene.py [path|repo|all]
python ~/Documents/GitHub/Uprava/tools/bib_hygiene.py --apply   # non-lossy only
python ~/Documents/GitHub/Uprava/tools/bib_hygiene.py --selftest
```

Script: [`Uprava/tools/bib_hygiene.py`](https://github.com/gasyoun/Uprava/blob/main/tools/bib_hygiene.py) (H1967 (Grok 4.5), M2S wave-2 thin-gate ports).

**Auto-apply (script `--apply`):** field-name typos (`jounal`→`journal`), empty-field flags for review. **Never auto-merged:** duplicate works under different keys, key renames, entry deletes — those stay propose-only.

## Judgment residue (after the report)

1. For each `dup_work_doi` / `dup_work_title` cluster: pick the richer entry, list `\cite` sites for the loser key, rewrite only with a green build.
2. Convention-drift keys: rename only when every cite site is updated in the same pass.
3. Persist the findings table when the run is non-trivial.

Argument `$ARGUMENTS`: empty/cwd · repo name · path · `all` (every `.bib` under `GitHub/`, skipping `runs/`/`.venv`/…).

_Dr. Mārcis Gasūns_
