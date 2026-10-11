---
name: "cologne-runbook-loop"
description: 'DEPRECATED alias removed 25-08-2026 - use /cologne-runbook-all. On Codex/Grok it runs as a workflow: /workflow cologne-runbook-all (watch /workflows); kept so older handoffs still resolve. Do NOT use this deprecated alias for new work — call /cologne-runbook-all directly; kept only so older handoffs naming cologne-runbook-loop still resolve.'

---

# cologne-runbook-loop

Use this skill when the user asks to run the migrated source command `cologne-runbook-loop`.

## Command Template


# DEPRECATED - use /cologne-runbook-all

## Default path (seconds — run this first)

The mechanical loop half (run the runbook scan, parse its report rows, iterate the UNPROCESSED rows per repo) is ported to a deterministic script (H5492) — dry-run row list by default, `--apply` to iterate per-repo applies:

```sh
python ~/Documents/GitHub/Uprava/tools/cologne_runbook_loop.py --help
# dry scan + parsed row list (default; read-only):
python ~/Documents/GitHub/Uprava/tools/cologne_runbook_loop.py
python ~/Documents/GitHub/Uprava/tools/cologne_runbook_loop.py --taxonomy dictionary
# real run (per-repo --apply loop over UNPROCESSED rows):
python ~/Documents/GitHub/Uprava/tools/cologne_runbook_loop.py --apply
```

LLM / judgment residue only after the script report exists

What stays the LLM gate: per-issue applicability — issue TYPE classification and every `needs_judgment` row the runbook reports. The loop only re-invokes the runbook's own planned mechanical writes (labels/milestones/projects); it never invents a type and never decides an issue's taxonomy.

_Created: 03-07-2026 · Last updated: 26-09-2026 (H5492: mechanical route ported)_

The cologne-runbook-loop skill was deleted from all harnesses 25-08-2026 (catalog token-waste sweep).
This stub exists only so old runbooks resolve. Run /cologne-runbook-all instead —
on Codex/Grok that is the workflow form `/workflow cologne-runbook-all` (watch `/workflows`).

_Dr. Mārcis Gasūns_
