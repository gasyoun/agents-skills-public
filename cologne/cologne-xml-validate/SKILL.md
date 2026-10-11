---
name: "cologne-xml-validate"
description: 'Standalone XML validation of one Cologne dictionary from csl-orig source (make_xml.py / generate_dict.sh, Windows fallbacks baked in). Before any csl-orig-touching commit/PR, after corrections, when a dict won''t parse. Triggers: validate DICT, xml check. Do not use for markup normalization or a full correction pipeline — that is /cologne-markup-fix or /cologne-correction-queue (which already validates internally); this is XML validation alone.'

---

# cologne-xml-validate

Use this skill when the user asks to run the migrated source command `cologne-xml-validate`.

## Command Template


# /cologne-xml-validate

_Created: 03-07-2026 · Last updated: 05-09-2026_

## Default path (seconds — run this first)

```sh
python ~/Documents/GitHub/Uprava/tools/cologne_xml_validate.py --dict <dict>
```

LLM / judgment residue only after the script report exists (or the script exits non-zero with a real defect).

Argument: dictionary code (e.g. `mw`, `ap90`, `pwg`) — or `all` to sweep every dict under `csl-orig/v02/`.

Step 4 of the correction workflow as a standalone: regenerate the dict's XML from `csl-orig/v02/<dict>/<dict>.txt` and confirm it parses. Read-only with respect to csl-orig; all outputs go to a scratch dir the wrapper cleans up itself.

**Mechanical wrapper, no LLM gate on the validate path.** The wrapper script
[`Uprava/tools/cologne_xml_validate.py`](https://github.com/gasyoun/Uprava/blob/main/tools/cologne_xml_validate.py)
(H2792, Sonnet 5 - ZJ xml-validate) already implements Phases 1–4 below — preflight, BOM check, generate + ET-parse-success
gate, and scratch cleanup. This skill is a thin caller:

```powershell
python ~/Documents/GitHub\Uprava\tools\cologne_xml_validate.py --dict <dict>
python ~/Documents/GitHub\Uprava\tools\cologne_xml_validate.py --all
python ~/Documents/GitHub\Uprava\tools\cologne_xml_validate.py --selftest   # mechanical self-check, no live pipeline
```

Exit 0 = all PASS, exit 1 = at least one FAIL, exit 3 = ENVIRONMENT-SKIP (named
preflight missing — python3/mako/`csl-websanlexicon` sibling). Report the
script's own per-dict PASS/FAIL/SKIP lines and summary line verbatim — do not
re-derive them. No commits, no comments, no issues — this skill only observes.

## What the wrapper does (reference, not to be re-derived by hand)

- **Preflight**: python3/python on PATH, `mako` importable, `csl-websanlexicon` sibling of `csl-pywork`, `csl-pywork/v02` present. `xmllint` is NOT available locally and is never chased — ET parse success is the validation signal.
- **BOM check**: first 3 bytes of `<dict>.txt` must not be `efbbbf`. A BOM masquerades as an `init_entries Error 2` / `<LEND>` parse error in `hw.py`.
- **Generate + validate**: shells out to `csl-pywork/v02/generate_dict.sh <dict> <scratch>` (the fuller pipeline behind `make_xml.py`); pass signal is **"All records parsed by ET"** in stdout/stderr.
- **Cleanup**: scratch output directory is removed after each dict (`--keep-scratch` to retain for debugging).

If the wrapper itself is unreachable (script missing/broken), fall back to the
manual Phase 3 recipe: `cd csl-pywork\v02\makotemplates\pywork && python make_xml.py <dict> <scratchpad>\tempparent`, same "All records parsed by ET" pass signal, then `Remove-Item -Recurse -Force <scratchpad>\tempparent\<dict>` — but treat that as an environment problem to report, not the normal path.

_Dr. Mārcis Gasūns_
