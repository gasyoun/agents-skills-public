---
name: "hook-wiring-audit"
description: 'Audit which wired hooks actually EMIT on a channel the harness reads, never-measured rows, false positives, whether token-compression deleted safety payload. Judgment — Opus 5 (claude-opus-5). Triggers: "hook wiring audit", "is this hook actually heard".'

---

# hook-wiring-audit

Use this skill when the user asks to run the migrated source command `hook-wiring-audit`.

## Command Template


# /hook-wiring-audit [hook | event-class | --unmeasured | --papercuts]

_Created: 12-08-2026 · Last updated: 26-08-2026_

Skill-mine **U42**, approved by MG on the H2323–H2591 sheet (12-08-2026) — [SKILL_CANDIDATES_CENSUS_H2323_H2591_12_08_2026.md](https://github.com/gasyoun/Uprava/blob/main/archive/SKILL_CANDIDATES_CENSUS_H2323_H2591_12_08_2026.md), built as [H2601](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H2601-Opus_claude-config_skill-build-u42-hook-wiring-audit_12.08.26.md). It wraps the tool that already exists — [`scripts/audit_hook_emission.py`](https://github.com/gasyoun/claude-config/blob/main/scripts/audit_hook_emission.py) — which had no skill, so the discipline around it was re-improvised across 6 sessions.

Argument: one hook, an event class (`Stop`, `SessionStart`, `PreToolUse`, …), `--unmeasured` to work the blind rows, or `--papercuts` to work the false-positive backlog.

## The founding fact

**A hook exiting 0 reaches the model through exactly ONE channel — `hookSpecificOutput.additionalContext`. Bare stdout is swallowed.** [H1817](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1817-Opus_claude-config_pretooluse-warn-surfacing-sweep_29.07.26.md) found ten hooks that ran, decided correctly, and spoke into the void for months. Only exit-2-plus-stderr (🛑) was ever heard.

Everything below follows from that: a hook is not "working" because it ran. It is working when a fixture proves **which channel carried its output**.

## The five questions this skill answers

| # | Question | Failure it catches |
|---|---|---|
| 1 | Does each wired row **emit on a channel the harness reads**? | H1817 mute channel |
| 2 | Is every wiring row **measured**? | unmeasured-as-passing |
| 3 | Does the test gate **actually run** the suites for what changed? | H2557 trigger-map blindness |
| 4 | Do the 🛑 guards fire only on **real** targets? | H2569 false-positive erosion |
| 5 | Does every command a hook **shows** carry the flags its tool needs? | H2709/H2714 — a shown snippet is a default, not documentation |

## Phase 1 — Measure, and read "unmeasured" correctly

```sh
python scripts/audit_hook_emission.py
```

**Unmeasured is not passing.** At the H2568 (Opus 5 — hook-emission fixtures for the 46 unmeasured wiring rows) baseline the audit measured **17 of 63** rows: 0 failing, **46 unmeasured** — meaning no fixture had ever provoked those hooks into speaking, so nothing knew which channel they used.

Worst-covered by class: **Stop · SessionStart · UserPromptSubmit · SessionEnd** — every ⚠️-advisory row. That is the sharp end. The personal [CLAUDE.md](https://github.com/gasyoun/claude-config/blob/main/CLAUDE.md) enforcement table splits hooks into 🛑/🔁 (mechanically enforced, prose may stay thin) and ⚠️ (a reminder, so **the prose is load-bearing**). That split is only sound if the ⚠️ warning arrives. **An unmeasured advisory row is a rule believed to be hook-backed while possibly being backed by nothing** — and the prose behind it may already have been thinned on that belief.

## Phase 2 — Fixtures that assert the channel

Per unmeasured row, build a payload that provokes a **real** emission from the hook's own decision logic — read the logic, don't guess an input.

**`Fail =` a fixture that asserts the hook *ran* without asserting *which channel* carried the output.** That is the H1817 defect re-shipped as a green test, and it is worse than no fixture because it retires the question.

Fixtures must exercise **the live wiring in [`settings.json`](https://github.com/gasyoun/claude-config/blob/main/settings.json)**, never a hand-maintained list — a parallel list drifts from the thing it claims to describe, and then the audit measures fiction.

Landing point: `0 unmeasured`, **or** an explicit `unmeasurable: <reason>` per residual row.

**Current state, measured 13-08-2026 (H2568 delivered):** `49 measured · 0 failing · 0 unmeasured · 14 unmeasurable`, from a 17/0/46/0 baseline. Every Stop, SessionStart and UserPromptSubmit row now carries a fixture that declares the channel it expects. The 14 residual rows are closed by *argument* rather than by fixture — legitimate only because each carries a written reason, and worth re-reading rather than trusting: the honest reasons are "the hook only writes a marker file and returns 0" or "provoking it would push to the real Uprava", never "hard to test". Partial coverage by event class is legitimate; **silent** partial coverage is not.

**Two lessons the fixtures themselves taught (H2568), reusable on the next blind row:**

- **The clean control is the half that finds bugs.** Three hooks parsed `git status --porcelain` at a fixed `line[3:]` while reading it through a `git()` helper that `.strip()`s stdout — which eats the leading status space of the FIRST line only. `stop_changelog_dirty_check` therefore nagged sessions that *had* written their changelog entry, because CHANGELOG.md sorts first in most repos. Nothing in the violating half could have shown that; the `FAIL-NOISY` verdict on the clean payload did. Fixed in [`hook_util.porcelain_path`](https://github.com/gasyoun/claude-config/blob/main/hooks/hook_util.py), pinned by [`test_hook_util.py`](https://github.com/gasyoun/claude-config/blob/main/hooks/test_hook_util.py).
- **A once-per-session INJECT has no "clean payload" in the warn sense.** Its silence contract *is* the marker, so the clean control is the same `session_id` a second time. If the marker does not take, the row is noise on every prompt — which is that class's actual failure mode, and the only one worth screening for.
- **Some rows need redirected STATE, not a different payload** (a runs journal holding a rival's live start; an estate tree carrying drift). Those hooks read no payload field that could carry the distinction, so the fixture schema takes `env_bad`/`env_good`, and the three hooks that lacked one grew the house `CLAUDE_TEST_*` override their twins already had. A fixture that instead seeds the REAL shared journal is not a fixture — it is a phantom live run a later session will believe.

## Phase 3 — Trigger map ≠ coverage map

[H2557](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H2557-Opus_claude-config_hook-test-gate-cross-cutting-blind-3-failing-suites_10.08.26.md): the pre-commit test gate reused `audit_hook_test_coverage.guard_for_test()` as its trigger map. That function deliberately reports suites whose filename doesn't name their subject as cross-cutting (`covers=None`) — **correct for coverage credit** (it prevents overstating one guard and hiding others) and **wrong as a trigger map**, because those suites then only ran when the suite file itself was staged.

Ten suites were affected, including four `test_pre_shell_guards_*` variants covering the single most-edited hook. Staging that hook ran **1 of its 5 suites**, and a commit shipped 3 red tests.

So: check that staging a hook implicates *every* suite that exercises it, and keep the two maps separate by intent.

## Phase 4 — Compression must not eat safety payload

[H2505](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H2505-Opus_claude-config_compress-sessionstart-hook-output_09.08.26.md) compressed SessionStart output by ~230 tokens/session. Two of those edits deleted load-bearing instructions: "**never delete** a live index lock — **RENAME** it, because on Windows the failed rename IS the liveness test" (deleting a live lock corrupts the index). Restoring both cost **~8 tokens** against the ~230 saved.

Rule: a token-compression pass over hook output is only safe behind tests that assert the **safety payload** survives, phrase by phrase. Trimming emphasis is fine; trimming the instruction is not. Measure the saving and the restoration cost separately so the trade is visible.

## Phase 5 — False positives erode the signal

[H2569](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H2569-Opus_claude-config_hook-command-targets-helper-5-false-positive-papercuts_10.08.26.md): 5 of 8 open hook papercuts shared one root cause — **a guard matching its pattern anywhere in the raw command string instead of at the command's target or argv position**. A `-m` commit message quoting a dangerous command got blocked; `gh api` got blocked because a PR *body* mentioned csl-orig.

Why it matters beyond friction: each false positive forces a literal override marker onto a **safe** command, and that is how `[destructive-git-ok]` becomes reflex-pasted — at which point the guard stops meaning anything and the real block gets waved through by muscle memory. **The cost is the erosion of the one signal that makes 🛑 hooks trustworthy.**

Fix shape: one shared `command_targets(cmd)` parser in [`hook_config.py`](https://github.com/gasyoun/claude-config/blob/main/hooks/hook_config.py) returning op + positional targets + flag values, consumed by every guard that needs it. Per papercut, a fixture pair: **the exact previously-blocked safe command (must now pass)** and **a purpose-built dangerous twin (must still block)**. Without the twin, a false-positive fix silently becomes a false negative — the worse failure.

## Phase 5b — Audit what the hooks SHOW, not only what they decide

```sh
python scripts/audit_injected_snippets.py --surface all --all
```

[H2709](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H2709-Opus_claude-config_close-suggestion-omits-tier_14.08.26.md) → [H2714](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H2714-Opus_claude-config_injected-snippet-flag-audit_14.08.26.md): hooks are audited as code, and the strings they inject are reviewed as prose, if at all — yet **a copy-pasteable command in an injected block has a higher execution rate than most functions in the repo.** One remedy line missing `--tier` produced **20.6 % of August's closes naming no executor**, for 17 days, while every prime suspect (the bulk closer, both Stop hooks) was innocent. Read every injected snippet as an **API**: each flag it omits is a default it sets.

Baseline 14-08-2026: **126 snippets · 60 files · 0 flagged**, over all three surfaces the tool now scans (H2735, Opus 5 — commands-md snippet flag sweep) — `--surface hooks` **13/8** (4 fixed in the H2714 pass) · `--surface commands` **55/34** (skills, their Codex twins, `references/`, `rules/`) · `--surface tools` **58/18** (string literals in `Uprava/tools/*.py`). Report-only, exit 0 always — a nag that blocks a commit gets switched off. `--self-test` asserts the extractor against a known-good multi-line fenced command before any count is trusted; `UPRAVA_TOOLS_DIR=<worktree>/tools` audits a fix that has not yet reached the shared Uprava checkout.

**Adjudicate, don't auto-trust.** A deliberately minimal example is not a defect, and `EXPECTED` is a curated map, not a parse of each tool's argparse — a newly added tool is silently unchecked until it gets a row. **When you add or reword a snippet in a hook, re-run this**; it is the cheap half of the pass.

**The extractor's own defects are the reusable lesson** — every one was *measuring the source instead of what the agent sees*: Python concatenates adjacent string literals, so a shown command spans source lines and a naive scan truncates at the first quote (33 of 33 first-run "findings"); and a docstring sentence naming a tool is not a command being shown. `test_*.py` is excluded for the same reason. The Markdown twin repeated it four more ways (H2735) — a fence anchored at column 0 skips every fence nested in a list, an inline span allowed to start on a fence backtick eats whole blocks, a `\``-escaped backtick truncates the span, and `--title "..."` reads as an ellipsis-elision — plus two inversions where the checker did not know its own tools' CLIs (`claim_handoff.py` takes the tier as a positional; `mint_handoff.py --batch` takes effort from the spec file). If an instrument does not reconstruct the bytes the model actually receives, it is measuring a different artifact than the one causing the behaviour.

## Phase 6 — Report and route

- `pytest hooks/` green + `python scripts/run_hook_tests.py` selection green.
- Before/after `audit_hook_emission.py` **and** `audit_injected_snippets.py` counts in the close — the measurement is the deliverable.
- Update [HOOKS_INDEX.md](https://github.com/gasyoun/claude-config/blob/main/HOOKS_INDEX.md) rows whose class or channel changed.
- If a ⚠️ row turns out to have been mute, **check whether its prose was thinned on the assumption it was hook-backed** and restore it — that is the H1812 (Opus 5 — claude-md hook-table block-vs-warn correction), not an optional extra.
- Residual friction → [`Uprava/tools/papercut.py`](https://github.com/gasyoun/Uprava/blob/main/tools/papercut.py); reusable caveats → [`/findings-append`](https://github.com/gasyoun/claude-config/blob/main/commands/findings-append.md).

## Hard rules

- **Assert the channel, never just the run.**
- **Unmeasured ≠ passing**; say the count out loud.
- **Fixtures come from live `settings.json` wiring**, not a parallel list.
- **Never fix a false positive without a dangerous twin** proving the block still works.
- **Never compress hook output without a test pinning the safety payload.**
- A ⚠️ hook found mute implicates the **prose** behind it too.
- **Every command a hook OR A SKILL shows is a default** — re-run `audit_injected_snippets.py --surface all` after touching any injected string, skill snippet, or tool help text. An adjudicated exemption is recorded next to the snippet as `snippet-audit: ok — <reason>` (reason mandatory) and is printed in every report, never silently dropped.

## Not this skill

| Situation | Use instead |
|---|---|
| Write/change a hook or permission in settings.json | built-in `update-config` |
| Keep a skill in sync across Claude/Codex | [`/claude-codex-skill-porter`](https://github.com/gasyoun/claude-config/blob/main/commands/claude-codex-skill-porter.md) |
| Which adopted skills never fire | [`/skill-effectiveness`](https://github.com/gasyoun/claude-config/blob/main/commands/skill-effectiveness.md) |
| A hook blocked you right now and you need to proceed | AGENT_INFRA_MACHINE_DEEP_MANUAL §5 recovery playbook |

_Dr. Mārcis Gasūns_
