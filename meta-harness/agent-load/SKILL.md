---
name: "agent-load"
description: 'One-command on/off for heavy PC load — handoff drains, roadmap batch drain, wisdomlib night scrape, watchdogs. Triggers: /agent-load, stop all drains.'

---

# agent-load

Use this skill when the user asks to run the migrated source command `agent-load`.

## Command Template


_Created: 15-08-2026 · Last updated: 05-09-2026_

# /agent-load — heavy agent load on/off

_Created: 31-07-2026 · Last updated: 05-09-2026_

Mechanical control plane for everything that overloads the PC at night/day:

| Load | Mechanism (darwin) | Mechanism (Windows) |
|---|---|---|
| Night quota-gated drain | launchd `com.gasyoun.uprava-drain-night` 03:00 | `sonnet-batch-drain-2200` + `UpravaDrainNightQuotaGated` |
| Roadmap batch drain | `roadmap_batch_drain.py schedule` | `roadmap-batch-drain-0300` |
| Sonnet handoff arm | `handoff_batch_drain.py schedule` | `handoff-drain-sonnet-daytime` |
| Opus handoff drain | opt-in (`--opus`) | `handoff-drain-opus-2200` (opt-in only) |
| Decision digest | launchd `com.gasyoun.uprava-decision-digest` 09:00 | `UpravaDecisionDigest0900` |
| Wisdomlib night scrape | n/a (Windows/D:\Tools estate) | `WisdomlibScrapeNightStart` 23:00–06:00 |
| Watchdog / morning report | — | 30m watchdog + 07:00 report |

**Script (source of truth):**
`~/.claude/scripts/agent_load_control.py`

## Invocations

| User says | Agent runs |
|---|---|
| `/agent-load off` · `stop all drains` · `PC overloaded` · `stop night` | `python ~/.claude/scripts/agent_load_control.py off` |
| `/agent-load on` · `turn drains back on` | Ask which arms, or use defaults below |
| `/agent-load on --all` | `python ~/.claude/scripts/agent_load_control.py on --all` |
| `/agent-load status` | `python ~/.claude/scripts/agent_load_control.py status` |

## Phase 0 — Always run the script (do not re-derive)

```sh
python ~/.claude/scripts/agent_load_control.py off
python ~/.claude/scripts/agent_load_control.py on --all
python ~/.claude/scripts/agent_load_control.py status
```

Partial **on** (recommended after an overload):

```sh
python ~/.claude/scripts/agent_load_control.py on --night --workers-night 1
python ~/.claude/scripts/agent_load_control.py on --night --roadmap
```

## What `off` does

1. darwin: `launchctl bootout` the drain/digest labels + deletes their plists; Windows: deletes the schtasks arms.
2. Writes `DO_NOT_RUN.flag` under `~/.claude/sonnet-drain`, `roadmap-drain`, `handoff-drain` — every drain driver honours the flag before spawning anything.
3. Logs every action to `~/.claude/agent-load/actions.jsonl` (loss-proof writer).

## What `on` does

1. Clears halt flags.
2. Re-arms only the selected arms through Uprava tools (`drain_night_launch.py schedule`, `roadmap_batch_drain.py schedule`, `handoff_batch_drain.py schedule`).
3. `--all` = night + roadmap + sonnet handoffs. **Opus is never in `--all`** (pass `--opus` explicitly).

## Agent behaviour

- User says overloaded / stop drains / no night → **`off` immediately**, then `status`, report one line.
- User says resume / turn on → prefer **`on` with explicit arms**; if they say "everything", use `--all`.
- Never invent a second stop mechanism; always go through this script.
- After `off`, do **not** re-arm until the user asks again (standing rule from 31-07-2026).

## Status table (relay to user)

Print script stdout. Summarise:

| Check | Good after `off` |
|---|---|
| Armed labels/tasks | none |
| Halt flags | `HALT` |

## Related (do not use these for emergency stop)

| Skill / tool | Role |
|---|---|
| `/drain` · `/roadmap-batch-drain` | Execute or arm roadmap units — not emergency stop |
| `handoff_batch_drain.py cancel` | One executor only |

## Where this is documented

| Surface | Path |
|---|---|
| This skill | `~/.claude/commands/agent-load.md` |
| Script | `~/.claude/scripts/agent_load_control.py` |
| Launch-box matrix | [STANDING_POLICY_LAUNCH_BOX_WIN_MAC_2026.md](https://github.com/gasyoun/Uprava/blob/main/docs/STANDING_POLICY_LAUNCH_BOX_WIN_MAC_2026.md) |

## Change log

Newest first. This section exists so the metadata line at the top can stay two dates long.

| Date | Change |
|---|---|
| 30-08-2026 | H3738 dual-platform rewrite: the original `agent_load_control.py` — deleted from the estate, Windows-schtasks-shaped — is replaced by a dual-platform script; darwin drives launchd bootout + halt flags, Windows keeps the schtasks arms |

_Dr. Mārcis Gasūns_