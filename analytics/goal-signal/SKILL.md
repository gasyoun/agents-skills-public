---
name: "goal-signal"
description: 'Thin wrapper running raw-signal /goal loops (G5, G14, … G68) via the committed goal_signal_run.py driver — cd + command + pass criteria, no re-derivation. Triggers: run goal G5, goal signal.'
---

# goal-signal

Use this skill when the user asks to run the migrated source command `goal-signal`.

## Command Template


# /goal-signal [G#|all-raw] [--list]

_Created: 28-07-2026 · Last updated: 26-08-2026_

Closes the 20 🔴 raw-signal goals in
[GOALS_SKILL_HOOK_COVERAGE_MATRIX.md](https://github.com/gasyoun/Uprava/blob/main/GOALS_SKILL_HOOK_COVERAGE_MATRIX.md)
(Finding 4 class): deterministic emitters already lived in
[GOALS_MANUAL.md](https://github.com/gasyoun/Uprava/blob/main/GOALS_MANUAL.md),
but every fresh session re-derived `cd` + command from prose. This skill is the
one-command surface; the commands themselves stay the goal's signal (never
replaced by a softer proxy).

**Driver-lane note (measured 26-08-2026):** the committed driver's
`all-raw` / `--list` lane currently carries **22** entries — the 20 covered here
plus convention-only guardrails G62/G63, which
[GOALS_SKILL_HOOK_COVERAGE_MATRIX.md](https://github.com/gasyoun/Uprava/blob/main/GOALS_SKILL_HOOK_COVERAGE_MATRIX.md)
marks ⬛ skill-less. Expect them in `all-raw` and `--list` output; they are not
part of this wrapper's covered registry table below.

**Argument:** a single `G#` (e.g. `G5`), a space-separated list (`G47 G48 G52`),
or `all-raw` (default when omitted). `--list` prints the registry without
running anything.

**Tier:** mechanical — Haiku 4.5 (`claude-haiku-4-5`) / Sonnet 5 (`claude-sonnet-5`).

## Phase 1 — Run the driver (do not re-derive)

From the org root (`GitHub/`):

```sh
python Uprava/tools/goal_signal_run.py G5
python Uprava/tools/goal_signal_run.py all-raw
python Uprava/tools/goal_signal_run.py --list
```

If the session is already inside a worktree of Uprava, use that worktree's
`tools/goal_signal_run.py` — same script, same registry. Prefer the script over
hand-typing the pytest/npm lines from the manual: the registry is the source of
truth for cwd + argv + pass substring, and it tracks the manual's Signal lines.

## Phase 2 — Report

Per goal print: **PASS / FAIL / SKIP** (repo not cloned), measured stdout tail,
exit code, and the `/goal` line from the manual. On FAIL: name the failing phase
and stop — do not brute-force retries past the goal's Cap without fixing the
underlying builder.

When used under a live `/goal` stop-condition, the evaluator re-checks the same
signal the script just ran; a green script run is evidence for the status-board
"Last verified" cell.

## Phase 3 — On red (fix, don't invent)

- **Fix the owning builder/source**, never relax an assertion or rewrite a lock
  count to green the gate (G39 pedagogy suites; G49 hub/MW/DCS lock).
- **Regenerate derived files** where the goal says so (G14 loci, G48 lemma
  summary, G50 app_data via import scripts) — never hand-edit the derived
  artifact to silence the validator.
- **G40 EN promote** is not this skill's job — the signal is
  `lang_parity_check` + empty `en_residual_keys` for already-promoted roots;
  the first `promote_en` run remains a deliberate act outside this wrapper.
- **G15 / G17 heavy builds** can take minutes; cap-respect means stop after one
  full red report, not a silent second full `npm run build`.

## Registry (raw goals)

| G# | Repo | Signal shape |
|---|---|---|
| G5 | ORS-FAQ | `pytest ors_faq/tests` (≥210) |
| G14 | csl-corrections | `build_correction_loci.py --selftest` |
| G15 | csl-atlas | `npm test` + review/feed validators + build |
| G17 | csl-observatory | `refresh_observatory.py --check-only` + observable build |
| G18 | csl-observatory | `data_index.py --check` + obs_t + build |
| G38 | SanskritGrammar | `pytest` + `npm run build` |
| G39 | kosha | 5 pedagogy pytest modules |
| G40 | SanskritLexicography | `lang_parity_check` + `en_residual_keys` empty |
| G47 | IndologyScholars | `validate_publication` + pytest (≥176) |
| G48 | VisualDCS | `validate_dcs_lemma_summary.py` |
| G49 | MWS | `root_crosswalk.py` lock 935/809/590/550 |
| G50 | BookIndex | `validate_content.py app_data.json` |
| G51 | RussianRamayana | `validate_data.py` 0 failed |
| G52 | csl-pyutil | `pytest` (≥24) |
| G59 | RuWritingStyles | `tools/validate_project.py` SUCCESS |
| G60 | SanskritKaraoke | `tools/validate_library.py` all verses valid |
| G61 | buhler-sanskrit-book | `npm test` (vitest ≥26) |
| G64 | Uprava | `audit_vote_hub_sheets --check` → VOTE HUB V11: PASS |
| G65 | Uprava | `<client>_private_separate verify` vs private sibling → PASS |
| G68 | SamudraManthanam | `pytest test_ai_spend_policy` (web/, ≥1) |

## When this isn't the right tool

- Goals that already have a real workflow skill (`/pwg-drain`, `/watcher-safe-commit`,
  `/handoff-registry-audit`, …) — invoke that skill, not this wrapper.
- Authoring or extending a goal's *definition* — edit
  [GOALS_MANUAL.md](https://github.com/gasyoun/Uprava/blob/main/GOALS_MANUAL.md)
  + `tools/goals_matrix_data.json`, then `goals_check.py --emit` (G32).
- Human-only or deploy-gated signals (ORS-FAQ real ladder conversion, money
  merge, scholar packets) — not goal-signal-shaped.

_Dr. Mārcis Gasūns_
