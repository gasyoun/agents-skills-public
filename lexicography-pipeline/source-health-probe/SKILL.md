---
name: "source-health-probe"
description: 'Probe external Sanskrit sources (DharmaMitra, VedaWeb, DCS…) lightly before fetching; log up/down/degraded in Uprava/SERVER_OUTAGES.md so sessions skip downed hosts. Haiku 4.5 (`claude-haiku-4-5`)/Sonnet 5 (`claude-sonnet-5`). Trigger: "probe the source".'
---

# source-health-probe

Use this skill when the user asks to run the migrated source command `source-health-probe`.

## Command Template


# /source-health-probe <host-or-url>

_Created: 14-07-2026 · Last updated: 05-09-2026_

## Default path (seconds — run this first)

```sh
python ~/Documents/GitHub/Uprava/tools/source_health_probe.py <host-or-url>
python ~/Documents/GitHub/Uprava/tools/source_health_probe.py --board-only
python ~/Documents/GitHub/Uprava/tools/source_health_probe.py --selftest
```

Script: [`Uprava/tools/source_health_probe.py`](https://github.com/gasyoun/Uprava/blob/main/tools/source_health_probe.py) ([H1967](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1967-Grok_Uprava_m2s-wave2-thin-gate-ports_30.07.26.md) (Grok 4.5 (`grok-4.5`) — M2S wave-2 thin-gate ports)). Reads board hits first, one HEAD/GET probe, classifies up/degraded/down, **never retries 429**.

## Judgment residue (log-before-abandon)

Update [`Uprava/SERVER_OUTAGES.md`](https://github.com/gasyoun/Uprava/blob/main/SERVER_OUTAGES.md) with a proper table row (Active / Recovered / permanent block) — model tier + exact version + rerun instruction. Optional script `--log` only appends a minimal HTML comment marker; the skill owns the human-readable row. Commit Uprava from a worktree.

_Dr. Mārcis Gasūns_
