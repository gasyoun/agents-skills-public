---
name: "skill-mine"
description: 'Mine session footprints (handoffs, commit history) for repeated improvised workflow shapes as skill candidates — ≥2-session bar, dedupe, ranked list + review sheet for human vote. Triggers: "skill mine", "/skill-mine", "what skills are we missing", "skill candidates from handoffs". Do not use to measure how existing skills perform — that is /skill-effectiveness; this mines session footprints for NEW skill candidates.'

---

# skill-mine

Use this skill when the user asks to run the migrated source command `skill-mine`.

## Command Template


# /skill-mine [executor-or-window]

_Created: 11-07-2026 · Last updated: 26-09-2026 (H5505: mechanical route ported)_

## Default path (seconds — run this first)

```sh
python3 ~/Documents/GitHub/Uprava/tools/skill_mine_tally.py --handoffs-dir ~/Documents/GitHub/Uprava/handoffs
git log --pretty=%s > /tmp/subjects.txt && python3 ~/Documents/GitHub/Uprava/tools/skill_mine_tally.py --subjects /tmp/subjects.txt
python3 ~/Documents/GitHub/Uprava/tools/skill_mine_tally.py --handoffs-dir ~/Documents/GitHub/Uprava/handoffs --subjects /tmp/subjects.txt --min-count 2
```

The script is the read-only Phase-1 tally: handoff files per repo-slug and per subject-token (archive/ subtree included), repeated subject n-grams from a `git log --pretty=%s` file, and one ranked repeated-shape gap list (rank · src · pattern · count · example IDs). It never edits anything and never rules on candidacy.

Encodes the footprint→skill census method that was re-derived five times, all
Fable 5 (`claude-fable-5`), early July 2026:
[H284](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H284-Fable_claude-config_worktree-gc-skill_07.07.26.md) — /worktree-gc from GitHub-history mining ·
[H295](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H295-Fable_claude-config_fable-skill-gaps_07.07.26.md) — Fable-fit skills from the git-history gap sweep ·
[H322](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H322-Fable_claude-config_skill-gaps-wave2_07.07.26.md) — wave-2 skills from the 2-month gap analysis ·
[H417](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H417-Fable_claude-config_fable-footprint-skills_09.07.26.md) — Fable-footprint skills audit (4 new, 3 updated) ·
[H435](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H435-Fable_Uprava_fable_footprint_skill_census_09.07.26.md) — footprint census → ranked candidate sheet.
Census candidate C2 in
[FABLE_FOOTPRINT_SKILLS_CENSUS.md](https://github.com/gasyoun/Uprava/blob/main/archive/FABLE_FOOTPRINT_SKILLS_CENSUS.md),
built via [H582](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H582-Fable_Uprava_skill-mine-skill-build_11.07.26.md) (Fable 5 (`claude-fable-5`)) — build the /skill-mine skill. Each prior pass re-invented the sweep sources, the bar, and the output shape; this skill is that method, written down once.

Optional argument: an executor-family filter (`fable`, `sonnet`, `opus`,
`haiku`, `codex`) or a date window; default = all executors since the newest
footprint census in Uprava. `Codex`, `Codex Sol`, `GPT-5*`, and exact
`gpt-5.6-sol` evidence normalize to the `Codex` family.

LLM / judgment residue only after the script report exists — the tally is mechanical; the ≥2-session reading, catalogue dedupe and ranking stay with the LLM/human census pass.

## Phase 1 — Sweep the three footprint sources (read-only)

Sweep: **(a) handoffs** — live 🔵/🟡 rows in
[Uprava/handoffs/README.md](https://github.com/gasyoun/Uprava/blob/main/handoffs/README.md)
plus terminal ✅ rows in [handoffs/REGISTRY_ARCHIVE.md](https://github.com/gasyoun/Uprava/blob/main/handoffs/REGISTRY_ARCHIVE.md)
(read BOTH halves; since the [H1798](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1798-Opus_Uprava_registry-split-active-vs-archived-rows_28.07.26.md) registry split — Opus 5, executed 04-08-2026, active rows vs archived rows — README holds only live rows and a one-file read silently sees an empty finished set),
grouping by workflow shape, not repo; **(b) commit subjects and trailers** across
the org clones; **(c) provenance docs** — completion/registry statements,
QUESTIONS_LOG, both FINDINGS hubs, ARTICLES.md, `.ai_state.md`, and standing
trackers. Record the distinct H### sessions behind each shape.

For every session store two columns: **intended executor** and **actual
executor**. Intended comes from the filename token and intended-executor line.
Actual attribution follows this precedence, stopping at the first affirmative
source: (1) completion/registry statement; (2) executed status in the handoff;
(3) commit provenance trailer; (4) filename token fallback. Never use
a catalogue sentence such as “Codex twin”, a skill path, or a portability note as
execution evidence. Preserve the exact observed model in a detail column while
normalizing aliases to the executor family for counts.

## Phase 2 — Shape extraction + the ≥2 bar

A candidate = the **same improvised discipline in ≥2 distinct sessions with no owning skill** (the bar `/crosswalk-build`, `/dh-memo`, `/gate-judge` were minted at), **plus one exception (MG 07-09-2026): a singleton is NOT rejected if its actual executor was Fable or Astra** — a top-tier session's improvised discipline is carried as a candidate even at freq 1, since the executor's judgment-heavy improvisations are exactly what the estate wants encoded. Singletons in other families are listed one line each and dropped — structurally-unique one-shots (resurrections, remakes) are explicitly rejected, with the rejection recorded so the next census doesn't re-litigate them. The record is the reject store [`data/review_rejects/skill-mine.json`](https://github.com/gasyoun/Uprava/blob/main/data/review_rejects/skill-mine.json), driven by [`tools/review_reject_store.py`](https://github.com/gasyoun/Uprava/blob/main/tools/review_reject_store.py): run `check --sheet <html>` on the new sheet before publishing (cards already ruled out are flagged as re-asks) and `append --decisions <json> --sheet <html> --apply` after the vote. Polarity matters — a reject on a *policy* card («don't build X?» → нет) means BUILD ([H2929](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H2929-Opus_Uprava_skill-mine-reject-polarity-fix_16.08.26.md), Opus 5 — inverted-polarity cards purge + detect).

## Phase 3 — Dedupe against the catalogue

Check every shape against the live catalogue (`~/.claude/commands/*.md` + [SKILLS_INDEX.md](https://github.com/gasyoun/github-spine/blob/main/SKILLS_INDEX.md)): covered / partial (name which skill owns the near-miss and what delta remains) / uncovered. A shape an existing skill *could* own with a one-paragraph update becomes an UPDATE item, not a new skill.

## Phase 4 — Rank, sheet, stop

Rank candidates by **frequency × token-savings-when-encoded × priority tier**. Persist the census as a dated doc in Uprava (or update the standing census doc), then emit a `/review-sheet` (approve/reject/defer per candidate + update items) and a GTD `@DO` vote row. **No skills are built in the census pass** — building happens only after the human vote (or an explicit "make them"), each approved item filled via `/handoff-mint`. When a build wave does run, every new skill also fires the three catalogue-parity loops: G29 (Codex twin), G34 (SKILLS_MANUAL_RU card), G35 (SKILLS_INDEX row) — in the same pass.

**Emitting the sheet means running `/review-sheet` Phase 3-bis too — the hub publish is not optional here** ([H2968](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H2968-Opus_Uprava_skill-mine-sheets-to-vote-hub_17.08.26.md), Opus 5 — publish skill-mine sheets to the vote hub by default, 17-08-2026). Copy the sheet to [`gasyoun.github.io/vote/sheets/`](https://github.com/gasyoun/gasyoun.github.io/tree/master/vote/sheets) under a short `skill_mine_h<from>_h<to>.html` name and put that URL in the sheet's [REVIEW_SHEETS_INDEX.md](https://github.com/gasyoun/Uprava/blob/main/REVIEW_SHEETS_INDEX.md) row, in the same pass that generates it; **commit the generator too** as `Uprava/tools/_gen_skill_mine_sheet_h<from>.py` (the local sheet is `review/uprava-skill-mine_h<from>-h<to>_<DD-MM-YY>_review.html`, gitignored). Skill-mine sheets are gitignored *and* deleted by `/decisions-apply` at apply time, so before the hub existed the local HTML was the only copy: `uprava-skill-mine_h2323-h2591_12-08-26` now has no local file, no hub copy and no committed generator, and its cards cannot be read at all (H2929 above) — which is also why the [H2923](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H2923-Opus_Uprava_skill-mine-reject-backfill_16.08.26.md) (Opus 5 — reject-store back-fill from surviving sheets) reached only 4 of 7 waves. **Fail = a skill-mine sheet whose index row carries a `file:///` link and nothing else.**

**Vote surface (standing rule, MG 26-08-2026):** a skill-mine slate of roughly ≤10 cards is voted **in chat, card by card** per `/review-sheet` Phase 3-ter — the hub copy is still published as the archival card text, but the chat answers are the official vote, transcribed to `review/weekly/archive/decisions_applied_<date>_<sheet>.md` before anything approved is built ([STANDING_POLICY_SMALL_SHEET_VOTES_IN_CHAT_2026-08-26.md](https://github.com/gasyoun/Uprava/blob/main/docs/STANDING_POLICY_SMALL_SHEET_VOTES_IN_CHAT_2026-08-26.md)). Larger slates stay on the hub flow.

## Gate that stays (LLM + human)

The script counts repetition, not candidacy. Whether a repeated shape is a REAL skill candidate — the same improvised discipline in ≥2 distinct **sessions** (never commits or subjects), the Fable/Astra singleton exception, and Phase 3 catalogue dedupe — stays with the LLM/human census pass. A row in the script's gap list is a lead, never a verdict.

Adoption is gated too: a voted-in candidate passes the [`/skill-fit-audit`](https://github.com/gasyoun/claude-config/blob/main/commands/skill-fit-audit.md) fit check against the hubs-derived org work profile before it syncs into the estate — census + drain alone do not admit a skill (wired H5790, 03-10-2026).

## Hard rules

- Read-only sweep agents; the census pass never edits skills.
- Every verdict (covered/candidate/rejected) carries its H### evidence;
  frequency counts sessions, not commits. Report intended and actual counts
  separately—filename-only counts are explicitly labelled fallback estimates.
- Dedupe before ranking; a candidate that duplicates a catalogue skill is a census defect.
- **Retention ([H3101](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H3101-Sonnet_Uprava_stale-docs-snapshot-archive-sweep_18.08.26.md), Sonnet 5 — archive the snapshot-series doc families, 19-08-2026):** the dated census doc and its
  `SKILL_MINE_H####_VOTE_APPLIED_*` twin are snapshots, not registries — once
  [SKILLS_INDEX.md](https://github.com/gasyoun/github-spine/blob/main/SKILLS_INDEX.md)
  reflects a wave's outcome, `git mv` that wave's files to
  [`archive/`](https://github.com/gasyoun/Uprava/blob/main/archive) in the
  same pass.

_Dr. Mārcis Gasūns_
