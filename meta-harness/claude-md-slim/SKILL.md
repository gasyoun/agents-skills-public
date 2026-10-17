---
name: "claude-md-slim"
description: 'Two modes: slim - token-slim CLAUDE.md by extracting repeated rules into on-demand skills; refresh - bring a STALE CLAUDE.md back to truth, drop dead pointers, regen the generated block. Triggers: "slim CLAUDE.md", "this CLAUDE.md is stale/lying".'

---

# claude-md-slim

Use this skill when the user asks to run the migrated source command `claude-md-slim`.

## Command Template


# /claude-md-slim [path] [--mode slim|refresh]

_Created: 01-08-2026 · Last updated: 03-09-2026_

**Two modes, one skill (U46, 19-08-2026).** They share a file and a set of
non-goals but answer different questions:

| Mode | Question | Failure it fixes |
|---|---|---|
| **`slim`** (default) | is this file too **expensive**? | static context overhead every session |
| **`refresh`** | is this file **true**? | an agent acting on a claim that stopped holding |

Pick `refresh` when the file is the right size but its facts have rotted — a
14-day-SLA breach, a pointer to a moved file, a "not yet built" note about
something that shipped weeks ago. Pick `slim` when the file is accurate and fat.
Running `slim` on a stale file is the common mistake: it makes false claims
shorter. **Refresh first, then slim.**

**Run seventeen times, not parked** (YK tip 29, org-adapted; status corrected 03-09-2026, [H3997](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H3997-Opus_claude-config_skills-usage-ranked-tierB-depth-pass_03.09.26.md)). This file carried "Parked for later execution" long after the skill had shipped seventeen sessions — a seven-session Grok 4.5 `slim` wave ([H2817](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H2817-Grok_Systema-Sanscriticum_claude-md-slim-tier0_15.08.26.md) Systema-Sanscriticum · [H2818](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H2818-Grok_github-spine_claude-agents-slim_15.08.26.md) github-spine · [H2819](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H2819-Grok_claude-config_claude-md-slim-always-on_15.08.26.md) claude-config · [H2820](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H2820-Grok_Uprava_guarded-claude-truth-refresh_15.08.26.md) Uprava · [H2821](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H2821-Grok_Uprava_dict-pipeline-claude-fill_15.08.26.md) Uprava · [H2822](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H2822-Grok_VisualDCS_claude-md-slim_15.08.26.md) VisualDCS · [H2975](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H2975-Grok_github-spine_claude-md-slim-residual_17.08.26.md) github-spine) and the ten-session `refresh` wave of 19-08-2026 (H3041–H3050, enumerated under Mode `refresh` below). That is precisely the stale **status claim** this skill's own R1 check below is written to catch, sitting in the skill that teaches it. The greenfield advice "keep CLAUDE.md tiny" is **wrong as a blanket** for multi-repo org spines. The right move is:

> Keep **always-on** facts that prevent data loss / wrong-repo rebuilds. Move **procedure** into skills that load only when invoked.

Argument: path to a `CLAUDE.md` / `AGENTS.md` (default: cwd's, then `~/.claude/CLAUDE.md` if reviewing global).

## Non-goals

- Do **not** gut org hubs (MEGABOOK pointers, SHARED_CODE, worktree guards, danger facts).
- Do **not** replace hooks with prose "reminders" that hooks already enforce.
- Do **not** invent a second copy of a skill that already exists under `commands/`.

## Phase 0 — Measure before editing

1. Byte size + approximate tokens (`chars/4` or `tiktoken` if available) of the target file and of every auto-loaded parent (`~/.claude/CLAUDE.md`, the org root — which is [github-spine/CLAUDE.md](https://github.com/gasyoun/github-spine/blob/main/CLAUDE.md), not a file at the GitHub clone root — and the repo's own).
2. [`/context`](https://github.com/gasyoun/claude-config/blob/main/commands/context.md) or statusline context % at a **fresh** session start — note the static overhead before any user turn.
3. List skills already covering sections of this file (`commands/*.md`, `skills/*/SKILL.md`).

Write the baseline into the reply as a table: `path | bytes | ~tokens | already covered by skill?`.

## Phase 1 — Classify every section (3 buckets)

| Bucket | Stays in CLAUDE.md | Moves to skill | Delete / demote |
|---|---|---|---|
| **Always-on safety** | worktree guards, danger seeders, csl-orig read-only, money-contour, encoding BOM | — | — |
| **Always-on routing** | hub table (one line per hub + URL), priority tiers one-liner | long hub essays | duplicate prose of the repo's own features hub (only [SanskritLexicography FEATURES_INDEX.md](https://github.com/gasyoun/SanskritLexicography/blob/master/FEATURES_INDEX.md) exists as one) |
| **Procedure** | — | multi-step workflows ("how to correct", "how to release") | if a skill already exists, **link it** and cut the prose |
| **Examples / history** | — | worked examples inside the skill | long incident narratives (point to FINDINGS §N instead) |
| **Already hook-enforced** | one-line pointer to HOOKS_INDEX | — | re-deriving the hook rule in prose |

Default bias: if the user must re-state it every session → always-on; if only for a named workflow → skill.

## Phase 2 — Extract one skill at a time

For each procedure section:

1. Check [`/prior-art`](https://github.com/gasyoun/claude-config/blob/main/commands/prior-art.md) / `commands/` for an existing skill — prefer **thin pointer**, not a new file.
2. If new: author `commands/<name>.md` with frontmatter `description` (trigger phrases), phases, non-goals.
3. Replace the CLAUDE.md section with 2–5 lines: what + when + `/skill-name`.
4. Keep full blob URLs to the skill file.

Cap one pass at **≤3 extractions** so the PR stays reviewable.

## Phase 3 — Verify

1. Fresh session: static context % should drop or hold; never rise.
2. Spot-run one extracted skill end-to-end on a dry topic.
3. Confirm danger / worktree / money rules still appear in the slim file (grep).
4. Changelog entry in the owning config repo — and check first whether it is a **queue repo** (claude-config · Uprava · Systema-Sanscriticum · kosha · csl-observatory as of 02-09-2026). There the entry is ONE FILE under `changelog_queue/` followed by [`/cut-release`](https://github.com/gasyoun/claude-config/blob/main/commands/cut-release.md); a direct bullet under `## [Unreleased]` is blocked by the pre-push hook. Only a non-queue repo takes the bullet inline. No force-push.

## Mode `refresh` — bring the file back to truth (U46, 19-08-2026)

Phases 0–3 above are the `slim` path. `refresh` replaces them with the following.
Ten Grok 4.5 sessions ran this one-repo-per-handoff on 19-08-2026 (H3041–H3050:
csl-devanagari · sanskrit-util · VisualDCS · csl-orig · csl-pywork ·
csl-websanlexicon · PWG · MWS · BookIndex · RuWritingStyles) after a seven-session
Grok 4.5 `slim` wave (H2817–H2822 plus H2975 — seven ids), with the procedure re-derived each
time. Six of the ten targets were Cologne satellites and four were not, which is
why this lives in the **org-generic** skill rather than in
[`/cologne-claude-md-fill`](https://github.com/gasyoun/claude-config/blob/main/commands/cologne-claude-md-fill.md) —
that skill **writes a first** `CLAUDE.md` for a Cologne satellite; this mode
**repairs an existing one, anywhere**. Reach for it when a file exists and lies.

### R1 — Verify every claim against the live tree, one by one

A `CLAUDE.md` is a set of assertions, so read it as a checklist and mark each:

- **Paths and pointers** — does the file/dir still exist at that path, on that
  branch, in that org? A moved file and a wrong-org blob URL fail identically for
  an agent. Check, do not assume.
- **Commands** — does the named script/flag still exist and still take that flag?
  Grep the tool, don't trust the prose.
- **Status claims** — "not yet built", "planned", "broken", "parked". These rot
  fastest and are the most expensive: an agent reads "not yet built" and rebuilds
  something that shipped. Grep for the *behaviour's data shape*, not the term the
  doc happens to use — a capability renamed or reimplemented under different
  wording reads as absent forever ([FINDINGS §497](https://github.com/gasyoun/Uprava/blob/main/FINDINGS.md)).
- **Counts and inventories** — "44 dictionaries", "16 guarded repos", "N tools".
  Recount; never carry a number forward on faith.
- **Prohibitions** — is the fence still real? A stale "no prod creds" style claim
  is worse than silence, because it stops correct work.

Record each verdict as `holds / moved / stale / dead`, with the evidence. This
table is the deliverable even for lines you do not change.

### R2 — Add what shipped since

Diff the repo's own record against the file: `git log --oneline <last-updated>..`,
the repo's `.ai_state.md` Completed section, its `CHANGELOG.md`, and any handoffs
naming it. Anything an agent would need on turn one — a new guard, a new canonical
tool, a new fence — is missing context, not an optional extra. Resist scope creep
in the other direction: this mode does not add *procedure*, it adds *facts*. New
procedure goes to a skill (that is `slim`'s job).

### R3 — Regenerate, never hand-patch, the generated block

If the file carries a `[generated-block: …]` marker, run its named generator
rather than editing inside the markers — a hand-patched block drifts back on the
next run and the drift check then fires on the innocent next session. Same rule as
[`/md-hygiene`](https://github.com/gasyoun/claude-config/blob/main/commands/md-hygiene.md)
Phase 2.6. Regenerate the twin (`AGENTS.md`) in the same pass so the two do not
disagree.

### R4 — Re-date and re-check

Bump `Last updated` (never `Created`), then re-run the freshness gate:
`python Uprava/tools/agent_docs_freshness.py --check`. A refresh that does not
move the gate did not happen.

### R5 — Batch honestly

A ten-repo wave is ten verifications, not one template applied ten times — the
whole value is per-repo truth. If you find yourself pasting the same paragraph
into the fourth repo, that paragraph belongs in the **org root** `CLAUDE.md` or a
skill, and each repo file should point at it instead. Say so in the report rather
than duplicating.

## Budget-trim note (U61, 11-09-2026)

The 07–08-09-2026 wave ([H4318](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H4318-Sonnet_Systema-Sanscriticum_claude-md-token-trim_07.09.26.md) → [H4343](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H4343-Sonnet_Systema-Sanscriticum_claude-md-token-trim-v2_08.09.26.md) re-mint, [H4319](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H4319-Sonnet_SanskritLexicography_claude-md-token-trim_07.09.26.md), [H4320](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H4320-Sonnet_Uprava_claude-agents-md-token-trim_07.09.26.md), [H4321](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H4321-Sonnet_claude-config_agents-md-add-claude-md-trim_07.09.26.md)) ran `slim` under a **token budget** rather than a section budget. Two additions that wave needed and this skill did not state:

- **Trim TO a target, not just "extract repeats".** Take the budget from the session ask (e.g. AGENTS.md ≤1100 approx-tokens, CLAUDE.md ≤3000) and stop when the measured figure (chars/4, Phase 0) is under it — the file's own routing value, not a fixed section count, decides what stays.
- **`NO_GIT_DATE` interplay (github-spine).** When the trim touches a file carrying the git-date convention, re-run the freshness checker in the same pass (`agent_docs_freshness.py --check`, R4) — a trim that strips the date line fails the gate on the next session's back. Sibling of the H4322 fix ([H4322](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H4322-Sonnet_github-spine_claude-md-git-date-fix_08.09.26.md)).

## Phase 4 — Stop conditions

Stop and report if:

- **`slim`:** the file is already mostly routing tables + skill pointers.
- **`slim`:** a proposed cut would remove a danger fact with no skill home.
- **`refresh`:** the file turns out to be accurate — say "verified, 0 changes" and
  stop. A refresh that finds nothing is a result, not a failure, and re-wording
  true prose to look busy is the anti-pattern.
- **`refresh`:** a claim cannot be verified either way (the tool it names is
  behind a paid gate, the host is down). Mark it `unverified` in the report with
  the reason; never silently delete a claim you merely could not check, and never
  promote a guess to a fact.
- **Either mode:** concurrent edit on the same CLAUDE.md (worktree / claim first).

## Resume starter (when a human wants this run)

```text
Read ~/.claude/commands/claude-md-slim.md and execute it on ~/.claude/CLAUDE.md then ~/Documents/GitHub/github-spine/CLAUDE.md (the org root; there is no CLAUDE.md at the GitHub clone root itself) — measure first, extract at most 3 procedures, do not gut safety/routing.
```

_Dr. Mārcis Gasūns_
