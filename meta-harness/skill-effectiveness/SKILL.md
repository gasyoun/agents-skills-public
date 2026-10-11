---
name: "skill-effectiveness"
description: 'Retrospective usage audit of adopted skills from session history — never-fired, fires-always (hook candidates), corrected-later (misfire proxy). NEVER-FIRED isn''t a kill signal. Read-only. Triggers: "skill effectiveness", "which skills never fire". Do not use to find brand-new skill candidates — that is /skill-mine; this measures adoption of skills that already exist.'

---

# skill-effectiveness

Use this skill when the user asks to run the migrated source command `skill-effectiveness`.

## Command Template


# /skill-effectiveness [window]

_Created: 23-07-2026 · Last updated: 05-09-2026_


## Default path (seconds — run this first)

```sh
python ~/Documents/GitHub/Uprava/tools/skill_effectiveness.py
```

LLM / judgment residue only after the script report exists (or the script exits non-zero with a real defect). Misfire-proxy / retire stays after that report.

### What this measurement CANNOT see (H3014, 19-08-2026 — read before quoting a never-fired verdict)

The script's numerator is **typed slash-lines only**. A skill fired through the **Skill tool** from the model's own listing, or reached by an agent **reading `commands/<name>.md` / `skills/<name>/SKILL.md` directly**, is typed by nobody and is invisible to this tally — and both routes dominate subagent work. Measured on the day the instrumentation hook was wired, **2 of 2 live invocations across two concurrent sessions were invisible** ([FINDINGS §495](https://github.com/gasyoun/Uprava/blob/main/FINDINGS.md)). [`hooks/posttooluse_playbook_invocation.py`](https://github.com/gasyoun/claude-config/blob/main/hooks/posttooluse_playbook_invocation.py) now records the tool-boundary route to `<claude>/playbook-invocations.jsonl`; read that ledger beside every run of this skill:

```sh
python ~/.claude/scripts/check_playbook_invocations.py --compare-history
```

Merging that ledger into the script is unfinished work. Until it lands, a low count here means **"rarely typed"**, never "rarely used" — the 23-08-2026 sections of [`SKILL_EFFECTIVENESS_LOG.md`](https://github.com/gasyoun/Uprava/blob/main/SKILL_EFFECTIVENESS_LOG.md) record this exact correction ("19-08 «silence» is a coverage gap, not breakage").

Slash-command surface for the org-native agent [`skill-effectiveness-analyzer`](https://github.com/gasyoun/claude-config/blob/main/agents/skill-effectiveness-analyzer.md) ([H1187](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1187-Sonnet_claude-config_subagent-fit-sa1-sa7-build_17.07.26.md) (Sonnet 5) — subagent-fit SA1–SA7 build; SA-7 = this analyzer). Pre-adoption is [`/skill-mine`](https://github.com/gasyoun/claude-config/blob/main/commands/skill-mine.md); at-adoption fit is [`/skill-fit-audit`](https://github.com/gasyoun/claude-config/blob/main/commands/skill-fit-audit.md); this skill answers **post-adoption usage**. Phases 1–2 (denominator + frequency tally) are arithmetic and now run through [`Uprava/tools/skill_effectiveness.py`](https://github.com/gasyoun/Uprava/blob/main/tools/skill_effectiveness.py) ([H2800](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H2800-Sonnet_Uprava_zj-skill-effectiveness-count_15.08.26.md) (Sonnet 5) — zj skill-effectiveness count port) instead of a hand-walk — the 23-07-2026 pass in [`SKILL_EFFECTIVENESS_LOG.md`](https://github.com/gasyoun/Uprava/blob/main/SKILL_EFFECTIVENESS_LOG.md) was the last manual run.

Argument: optional date window (`30d`, `90d`, `since:YYYY-MM-DD`); default = all history present in `history.jsonl`, with the actual span stated.

**Tier:** mechanical tally + light judgment on workhorse/hook candidates — Sonnet 5 (`claude-sonnet-5`). Read-only.

## Phase 0 — Policy (binding)

MG ruling 23-07-2026 ([H1531](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1531-Opus_claude-config_skill-shortlist-build-effectiveness_23.07.26.md) (Opus 4.8 (`claude-opus-4-8`)) — skill shortlist build + effectiveness policy):

1. **NEVER-FIRED ≠ bad skill.** Absence of invocations means the trigger has not occurred yet, or the skill is rare-but-load-bearing. Do **not** recommend deletion from a never-fired row.
2. **Kill/retire decisions wait ~6 months** after adoption (or after the last catalogue touch), not after a single audit pass. The audit is triage input for a human `/weekly-review`-style glance, not a retirement verdict.
3. **WORKHORSE** skills may be flagged as candidates for a hook/default — that is the productive outcome of high frequency, not a complaint.

## Phase 1 — Denominator

**Run the H2800 mechanical port instead of hand-walking this** —
[`Uprava/tools/skill_effectiveness.py`](https://github.com/gasyoun/Uprava/blob/main/tools/skill_effectiveness.py):

```
python ~/Documents/GitHub/Uprava/tools/skill_effectiveness.py [--window 30d|90d|since:YYYY-MM-DD]
```

It builds the catalogue from, in order of preference: (1) live `~/.claude/commands/*.md`
(+ `~/.claude/skills/*/SKILL.md` if present), (2) git-backed `claude-config/commands/`
when the live copy looks stripped (FINDINGS §48 / [H1433](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1433-Opus_Uprava_subagent-triage-coverage-wrong-tree-fix_21.07.26.md) (Opus 4.8 (`claude-opus-4-8`)) — subagent triage coverage wrong-tree fix, the path-home trap) — and prints the
count + which tree it used. Record that line in the report.

## Phase 2 — Frequency signal

Same invocation covers this phase: the script discovers **every** live `history.jsonl`
this host's harness could have written to — not just `Path.home()/.claude` (on Windows
agent hosts this is often `<claude-tools>\profiles\<profile>\.claude\history.jsonl` in
addition to the real home, resolved the same way as `goals_check.py` / [H886](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H886-Opus_Uprava_g32-goals-check-profile-dir-resolver_13.07.26.md) (Opus 4.8 (`claude-opus-4-8`)) — G32 goals-check profile-dir resolver, via
`tools/claude_dir.py`) — streams each one line-by-line (never loads a full file into
context), and tallies `display` values starting with `/<skill-name>`. It prints the
hit table, the never-fired count, and the file's true first/last timestamp span. Use
`--selftest` to sanity-check the script itself before trusting a live run.

## Phase 3 — Buckets

| Bucket | Definition (the script's own thresholds) | Action in the report |
|---|---|---|
| **WORKHORSE** | ≥10 hits | Flag as hook/default candidates with the skill name + why. |
| **MID** | 3–9 hits | Counts only — the band the script prints between workhorse and rare. Nothing to act on unless a MID skill sits on a daily path. |
| **RARELY-FIRED** | 1–2 hits | List with counts. Human glance only. |
| **NEVER-FIRED** | 0 hits in window | List with one-word note: `plausibly-rare` (default) or `check-trigger` if the description looks like a daily path. **No drop recommendation**, and never without the blindness caveat above. |
| **ORPHAN HITS** | fired, but absent from the current catalogue | A rename or a retirement that left callers behind. Resolve the name against the catalogue; never report it as usage of a live skill. |
| **MISFIRE-PROXY** | co-occurrence in FINDINGS / feedback / papercuts with “wrong/reverted/stale/false positive” | Quote context; label **PROXY** — confirm before acting. |

The first five are exactly what `report()` prints (`WORKHORSE (>=10 hits)` · `MID (3-9 hits)` · `RARELY-FIRED (1-2 hits)` · `NEVER-FIRED (0 hits)` · `ORPHAN HITS`); MISFIRE-PROXY is the one bucket the script cannot compute and you assemble by hand.

## Phase 4 — Fan-out option

For a large catalogue, spawn the read-only `skill-effectiveness-analyzer` agent with this skill’s policy block pasted into the prompt so it does not invent kill recommendations. Parent skill owns the final report shape and the 6-month rule.

## Phase 5 — Report + optional durable note

Emit:

```
WINDOW: <range> (history.jsonl span: …)
CATALOGUE SIZE: <N> (source tree)
WORKHORSE (N): …  → hook/default candidates
MID (N): …
RARELY-FIRED (N): …
NEVER-FIRED (N): … (measures TYPED invocations only — see the caveat above)
ORPHAN HITS: … (fired but not in the catalogue — renamed/retired?)
MISFIRE-PROXY: … (PROXY — confirm)
POLICY: no kills from this pass; revisit retirement ≥6 months post-adoption
```

Optional: append a dated section to [`Uprava/SKILL_EFFECTIVENESS_LOG.md`](https://github.com/gasyoun/Uprava/blob/main/SKILL_EFFECTIVENESS_LOG.md) — it already carries dated sections back to 23-07-2026, so append, never start it fresh — so the next audit has a baseline. Do **not** edit skill files, memory, or the catalogue from this pass.

## Hard rules

- Read-only: no skill deletions, no frontmatter edits, no registry flips.
- Never treat zero invocations as evidence the skill should go.
- Never report a proxy hit as a confirmed defect.

## When this isn’t the right tool

- “What skills are we missing?” → [`/skill-mine`](https://github.com/gasyoun/claude-config/blob/main/commands/skill-mine.md)
- “Does marketplace skill X fit us?” → [`/skill-fit-audit`](https://github.com/gasyoun/claude-config/blob/main/commands/skill-fit-audit.md)
- “Is skill file Y defective (paths, frontmatter)?” → [`/md-hygiene`](https://github.com/gasyoun/claude-config/blob/main/commands/md-hygiene.md) for the doc contract and [`Uprava/tools/skills_manual_check.py`](https://github.com/gasyoun/Uprava/blob/main/tools/skills_manual_check.py) / [`verify_skill_frontmatter.py`](https://github.com/gasyoun/Uprava/blob/main/tools/verify_skill_frontmatter.py) for frontmatter — not this. (There is no `skills-audit` command; the name this line used to give resolves to nothing.)

_Dr. Mārcis Gasūns_
