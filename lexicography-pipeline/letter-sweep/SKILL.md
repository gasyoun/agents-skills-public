---
name: "letter-sweep"
description: 'Byte-safe character-substitution sweep enforcing a style-guide letter rule (the ё→е ban shape): scope by git grep → script swap where EVERY changed file re-derives from HEAD by pure substitution (0 non-letter diffs) → move literal pins from tests along with the copy → proper-name/quoted-text exclusions listed for a human ruling, never changed silently → land via /watcher-safe-commit where a watcher is live → prod smoke grep count = 0. Triggers: "remove ё everywhere", "style guide letter ban", "letter sweep". Tier: Fable/Sonnet.'

---

# letter-sweep

Use this skill when the user asks to run the migrated source command `letter-sweep`.

## Command Template


# /letter-sweep <letter-or-rule> [repo]

_Created: 01-10-2026 · Last updated: 01-10-2026_

Distills the ё-sweep wave — 3 sessions, all Fable 5.1 (`claude-fable-5-1`) ✅ (H5206: style-guide §8 ban + 1 letter; H5234: 25 student-facing files, 42 swaps; H5242: wider — views 165 files/518 swaps + Console/Commands 171/818, 29 test pins moved, 7 exclusions), 20–22-09-2026, landing via `/watcher-safe-commit` on Systema-Sanscriticum. Approved **C56** in the skill-mine delta census ([SKILL_CANDIDATES_CENSUS_H5156_H5567_01_10_2026.md](https://github.com/gasyoun/Uprava/blob/main/SKILL_CANDIDATES_CENSUS_H5156_H5567_01_10_2026.md)), MG chat vote 10/10 approve 01-10-2026.

The discipline this owns: **a letter ban is mechanical ONLY with a proof of mechanicalness — every diff must be pure substitution, or the sweep is an edit and belongs to a judgment skill.**

## Phase 1 — Scope + rule

- The rule cites its source (the ё ban = §8 of `SAMSKRTE_SAMSKRTAM_EDITORIAL_STYLE_GUIDE_2026.md`); a letter rule without a written source is a preference, not a sweep.
- Enumerate scope by `git grep -l ё` (both cases ё/Ё→е/Е); report estate-wide counts for a human wider-sweep decision before crossing repo boundaries.

## Phase 2 — Swap with proof

- Script the swap so each changed file re-derives from HEAD by PURE substitution — **0 non-letter diffs**. If a file needs anything beyond the character swap, it drops out of the sweep into a judgment lane.
- This proof is what makes the sweep safe on a watcher-afflicted repo and safe to review: the reviewer checks the rule, not 518 individual hunks.

## Phase 3 — Pins + exclusions

- Pre-grep tests/ for every ё-literal pin BEFORE swapping; move each assertion with the copy (a pin left behind is a red suite; a pin silently updated is a hidden behavior change — both are defects).
- Proper names, quoted text, and citation-verbatim strings are NOT swept — list them as exclusions for a human ruling (H5242 carried 7), never change them silently.

## Phase 4 — Land + smoke

- Land via [`/watcher-safe-commit`](https://github.com/gasyoun/claude-config/blob/main/commands/watcher-safe-commit.md) where a watcher process reverts uncommitted changes (Systema is watcher-afflicted).
- Prod smoke: `ssh` grep count of the banned letter on the live surface = 0, receipt in the PR body.

## Dedupe fence

- [`/ru-copy-pass`](https://github.com/gasyoun/claude-config/blob/main/commands/ru-copy-pass.md) — register-matched RU rewrite (judgment editing); a letter sweep rewrites nothing.
- [`/litredaktor`](https://github.com/gasyoun/claude-config/blob/main/commands/litredaktor.md) — de-AI-ify prose pass; different failure class entirely.

_Dr. Mārcis Gasūns_
