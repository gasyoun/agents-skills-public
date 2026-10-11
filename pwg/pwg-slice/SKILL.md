---
name: "pwg-slice"
description: 'Alias of /pwg-drain — resume/run a PWG→RU per-root/per-window translation slice on the current live lane; kept so queued runbooks naming pwg-slice still resolve. Triggers: next PWG slice, resume the translation run. Do not use as the live mechanism itself — this is a compatibility alias only; run /pwg-drain directly for new work.'

---

# pwg-slice

Use this skill when the user asks to run the migrated source command `pwg-slice`.

## Command Template


# /pwg-slice → alias of /pwg-drain

_Created: 03-07-2026 · Last updated: 05-09-2026_

The pwg-slice skill was deleted from all harnesses 25-08-2026 (catalog token-waste sweep).
This stub exists only so old runbooks resolve. `/pwg-slice` resolves to **/pwg-drain** and
nothing else — there is no separate slice mechanism: read
`~/.claude/commands/pwg-drain.md` and follow it (Gate 0 via **/pwg-live-gate** → cost
preflight → **/pwg-bounded-run** → **/pwg-window-close**).

_Dr. Mārcis Gasūns_
