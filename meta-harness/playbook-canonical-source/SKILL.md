---
name: "playbook-canonical-source"
description: 'Treat the Claude command .md as a playbook''s ONE canonical source - derive Codex/Grok twins; retirement loop instruments real invocations, applies waves, re-baselines tri-harness drift. Triggers: "generate the Codex/Grok twin". Opus 5 (claude-opus-5) tier.'

---

# playbook-canonical-source

Use this skill when the user asks to run the migrated source command `playbook-canonical-source`.

## Command Template


# /playbook-canonical-source [playbook | --wave]

_Created: 19-08-2026 · Last updated: 26-08-2026_

Built from the [H3148 (Opus 5) — skill-mine census H2592–H3146](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H3148-Opus_Uprava_skill-mine-h2592-h3146_19.08.26.md) (candidate **C36**, 5 sessions, approved by
MG 19-08-2026): [H3014 (Opus 5) — playbook invocation instrumentation](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H3014-Opus_claude-config_playbook-invocation-instrumentation_17.08.26.md) · [H3015 (Sonnet 5) — playbook retirement evidence census, 0 of 235 candidates passed all three conjuncts](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H3015-Sonnet_Uprava_playbook-retirement-evidence-census_17.08.26.md) · [H3016 (Opus 5) — playbook retirement apply wave 1, `/encoding-bom-audit` retired](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H3016-Opus_claude-config_playbook-retirement-apply-wave1_17.08.26.md) · [H3017 (Sonnet 5) — playbook tri-harness drift baseline + Grok twin auditor](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H3017-Sonnet_claude-config_playbook-triharness-drift-baseline_17.08.26.md) · [H3018 (Opus 5) — playbook canonical-source generator](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H3018-Opus_claude-config_playbook-canonical-source-generator_17.08.26.md).

## The one idea

A playbook has **three** homes — Claude `commands/<name>.md`, Codex
`~/.codex/skills/<name>/SKILL.md` + `agents/openai.yaml`, Grok `~/.grok/…` — and
today a human or agent edits all three. Every existing tool in this space
*detects* the resulting drift; none *prevents* it. The move is to make the Claude
`.md` canonical and **generate** the other two, so drift becomes structurally
impossible rather than perpetually audited.

**Distinct from its neighbours, deliberately:**

| Skill | Owns | Why it cannot own this |
|---|---|---|
| [`/claude-codex-skill-porter`](https://github.com/gasyoun/claude-config/blob/main/commands/claude-codex-skill-porter.md) | reconciling **two** hosts after the fact | fixes drift; does not remove the cost that creates it — and Grok is a third host it never sees |
| [`/skill-effectiveness`](https://github.com/gasyoun/claude-config/blob/main/commands/skill-effectiveness.md) | read-only usage audit | explicitly refuses to be a kill signal (MG 23-07-2026, ~6-month bar), so it cannot *apply* a retirement |
| [`/skill-fit-audit`](https://github.com/gasyoun/claude-config/blob/main/commands/skill-fit-audit.md) | at-adoption ruling | says nothing about what happens to a playbook afterwards |
| [`/skill-mine`](https://github.com/gasyoun/claude-config/blob/main/commands/skill-mine.md) | pre-adoption census | the other end of the same lifecycle |

## What already exists — call it, don't rebuild it

- [`scripts/audit_codex_twins.py`](https://github.com/gasyoun/claude-config/blob/main/scripts/audit_codex_twins.py)
  and [`scripts/audit_grok_twins.py`](https://github.com/gasyoun/claude-config/blob/main/scripts/audit_grok_twins.py) —
  drift **detection**, both hosts. The 17-08 baseline audited 262 Codex + 103 Grok
  playbooks and found 21 distinct defects.
- [`scripts/check_playbook_invocations.py`](https://github.com/gasyoun/claude-config/blob/main/scripts/check_playbook_invocations.py)
  and the PostToolUse emitter — the **real** invocation ledger.
- [`scripts/generate_playbook_twins.py`](https://github.com/gasyoun/claude-config/blob/main/scripts/generate_playbook_twins.py)
  (H3018) — Claude command → Codex `SKILL.md` + `agents/openai.yaml` + Grok twin.
  Refuses `SUSPECT_DIRECTION` overwrites. CI: `.github/workflows/twin-drift.yml`.

## Phase 1 — Count invocations at the TOOL boundary, never at the prompt

The number every retirement decision rests on. Get it wrong and you retire live
playbooks.

**`history.jsonl` slash-lines count typing, not invocation.** A model calling the
Skill tool, or an agent reading `commands/<name>.md` directly, invokes a playbook
with nobody typing anything — and on the day the emitter was wired, **2 of 2 real
invocations were invisible** to the slash-line tally
([FINDINGS §495](https://github.com/gasyoun/Uprava/blob/main/FINDINGS.md)). So:

- Read `~/.claude/playbook-invocations.jsonl` (the PostToolUse ledger), not the
  slash-line tally, and treat any older "never fired" verdict as an artefact of
  the instrument until re-measured.
- **Scope the comparison to sessions the ledger actually saw.** The ledger is
  young; an all-time side-by-side flatters it for free.
- **Precision over recall**: a `Read` counts only when an ancestor path segment is
  a harness config tree (`.claude` / `.codex` / `.grok` / `claude-config`). A repo
  that happens to keep `commands/deploy.md` is not a skill catalogue, and a false
  *fired* corrupts the decision silently while a false *never-fired* is visible.

## Phase 2 — Retirement census: every conjunct, or no retirement

A playbook is retired only when **all** conjuncts hold at once — never on
never-fired alone. H3015 scored 235 candidates and **0** passed, which is the
expected shape: this gate is meant to refuse.

1. **Not invoked** in the ledger's window, by the Phase 1 count.
2. **Past the age bar** — ~6 months (MG 23-07-2026). Younger silence is not evidence.
3. **Superseded or dead** — its job is owned by another playbook, or the thing it
   operated on no longer exists.
4. **No inbound references** — `SKILLS_INDEX.md`, `SKILLS_MANUAL_RU.md`, org
   `CLAUDE.md` files, open handoffs.

Report the score per conjunct, including for candidates that failed. A census that
prints only the retirements hides the fact that the gate did its job.

## Phase 3 — Generate the twins from the canonical

Claude `.md` is canonical. Generate Codex `SKILL.md` + `agents/openai.yaml` and the
Grok twin from it; never hand-edit a generated twin — a hand-edit dies silently at the
next generation (observed live: the `fruit` twin's 97-line hand-maintained body was
replaced by its generated stub mid-session, 23-08-2026). As of 26-08-2026, **242 of
274** Codex twins are generated stubs. The writer is
[`scripts/generate_playbook_twins.py`](https://github.com/gasyoun/claude-config/blob/main/scripts/generate_playbook_twins.py)
(H3018) — call it, do not write a second generator:

```sh
python ~/.claude/scripts/generate_playbook_twins.py <name>
python ~/.claude/scripts/audit_codex_twins.py --skill <name>
python ~/.claude/scripts/audit_grok_twins.py --skill <name>
```

Both auditors must exit 0 on that skill. `--all` generates every `commands/*.md`
and skips (does not overwrite) any richer existing twin.

**The one case that needs judgment, not codegen:** `SUSPECT_DIRECTION` — a Codex or
Grok bundle **richer** than its Claude counterpart. The generator **refuses** to
overwrite it (exit 2 for a named skill). Blindly repointing a richer bundle at the
thinner canonical silently degrades a live skill on every session. When the twin is
richer, the fix is to **lift its content into the canonical first**, then generate.
Stop and say so rather than generating over it.

After generating, re-run both auditors. A generator whose output still fails its
own auditor is not done. CI (`.github/workflows/twin-drift.yml`) generates into a
temp tree and fails the job on any finding.

## Phase 4 — Re-baseline and wire the gate

- Re-run the tri-harness drift baseline and record the new defect count against
  the previous one; a wave that does not move the number did not happen.
- The detection scripts are on-demand only. Wire them into CI so drift is caught
  at push rather than at the next census — that is the difference between
  *detected* and *prevented*.
- **Retirement is not deletion.** A retired playbook loses its `SKILLS_INDEX.md`
  row and gains a `retired:` prefix, so the name is not silently re-used and the
  history stays readable.

## Phase 5 — Catalogue parity, same pass

Any playbook this skill creates, retires, or renames fires all three loops
immediately: **G29** Codex twin · **G34** `SKILLS_MANUAL_RU.md` card · **G35**
`SKILLS_INDEX.md` row. That is the whole point — a canonical source that leaves its
own index stale has reintroduced the drift it exists to remove.

## Evidence required

- [ ] Invocation counts from the tool-boundary ledger, with the window stated
- [ ] Per-conjunct retirement scores, including the failures
- [ ] Generated twins + both auditors re-run to a zero drift count
- [ ] Any `SUSPECT_DIRECTION` case named, with the direction it was resolved in
- [ ] G29 / G34 / G35 done in the same pass

_Dr. Mārcis Gasūns_
