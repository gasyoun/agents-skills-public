---
name: "retired-encoding-bom-audit"
description: "Stream a data-export tree and report each text file's BOM state + encoding + size as a table, flagging inconsistencies within a sibling group (e.g. key1 has a BOM, key2 doesn't), so the \"BOM is inconsistent — check head -c 3 before editing, preserve on write\" rule becomes auditable instead of manual. Read-only. Use for \"audit the BOMs\", \"which files have a BOM\", \"check encoding across HeadwordLists\", or before a bulk transform of exported text files. Haiku 4.5 (claude-haiku-4-5), Sonnet 5 (claude-sonnet-5) escalation."
---

# retired-encoding-bom-audit

Use this skill when the user asks to run the migrated source command `retired-encoding-bom-audit`.

## Command Template

_Created: 14-07-2026 · Last updated: 29-08-2026_

---
description: Stream a data-export tree and report each text file's BOM state + encoding + size as a table, flagging inconsistencies within a sibling group (e.g. key1 has a BOM, key2 doesn't), so the "BOM is inconsistent — check head -c 3 before editing, preserve on write" rule becomes auditable instead of manual. Read-only. Use for "audit the BOMs", "which files have a BOM", "check encoding across HeadwordLists", or before a bulk transform of exported text files. Haiku 4.5 (Codex-haiku-4-5), Sonnet 5 (Codex-sonnet-5) escalation.
argument-hint: <tree-or-dir> [--fix-none]
---

# /encoding-bom-audit

## Default path (seconds — run this first)

```sh
python <home>/Documents/GitHub/Uprava/tools/encoding_bom_audit.py [dir]
python <home>/Documents/GitHub/Uprava/tools/encoding_bom_audit.py --selftest
```

Script: [`Uprava/tools/encoding_bom_audit.py`](https://github.com/gasyoun/Uprava/blob/main/tools/encoding_bom_audit.py) (H1967). Streaming first-3-bytes only — never slurp multi-MB exports. Read-only (never rewrites). Default tree: `SanskritLexicography/HeadwordLists` when present.

## Residue

- Persist the table when the run covers a full tree (`/log-tables` or repo `RESULTS_LOG.md`).
- Any actual BOM fix is a separate human-approved edit that preserves the file's intent (AGENTS.md Encoding rule).

_Dr. Mārcis Gasūns_
