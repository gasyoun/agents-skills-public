---
name: "skill-to-script-port"
description: 'Consolidate an LLM fan-out skill into ONE deterministic config-driven script - judgment-gated parts stay LLM; --dry-run-default idempotent script, equivalence proof, rewire the skill .md to CALL it. Triggers: "port this skill to a script", "de-LLM this skill". Do not use to author a brand-new skill from scratch — that is /skill-creator; this consolidates an EXISTING LLM fan-out skill into a deterministic script. Mode --residue-port: move a skill''s zero-judgment residue into a committed tools/*.py of the CONSUMING repo for cross-harness (opencode/codex/grok) execution.'
---

# skill-to-script-port

Use this skill when the user asks to run the migrated source command `skill-to-script-port`.

## Command Template


# /skill-to-script-port <skill-or-skill-family> [--consolidate]

_Created: 17-07-2026 · Last updated: 07-09-2026_

Four sessions independently re-derived the same move: a skill whose per-item work is
**deterministic** was paying agent tokens per item, and got collapsed into one script.
[H556](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H556-Sonnet_Uprava_cologne-batch-deploy-consolidation_11.07.26.md)
(Sonnet 5 (`claude-sonnet-5`) — cologne-batch-deploy consolidation)
folded seven `cologne-*-all` fan-outs into
[`cologne_batch_deploy.py`](https://github.com/gasyoun/Uprava/blob/main/tools/cologne_batch_deploy.py);
[H835](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H835-Opus_claude-config_skill-model-ver-fix-llm-to-py_13.07.26.md)
(Opus 4.8 (`claude-opus-4-8`) — skill model-ver fix LLM→py)
extracted `handoff_close.py` + `mdx_coverage.py`;
[H929](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H929-Opus_claude-config_docx-converter-tool-extraction-and-handoff-trim_14.07.26.md)
(Opus 4.8 — docx-converter tool extraction + handoff trim)
pulled ~600 lines of inline Python out of `docx-to-md.md`;
[H930](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H930-Opus_claude-config_hook-security-git-c-fail-opens-dead-guards-and-new-guard-hooks_14.07.26.md)
(Opus 4.8 — hook-security git-c-fail dead guards + new guard hooks)
hardened the guard hooks the same way. This skill is that move, encoded once.

Argument: a skill name (`/cologne-deploy-all codeql`), a family description ("all the
`*-all` deploy skills"), or nothing — infer the target from the conversation.
`--consolidate` signals **several** skills are expected to collapse into one script
(the H556 shape); omit it for a single-skill extraction (the H835/H929 shape).

**Tier:** judgment — Opus 4.8 (`claude-opus-4-8`). The mechanical-vs-judgment triage
in Phase 2 is the whole skill; getting it wrong either strands real judgment inside a
dumb script or leaves tokens burning on a set-difference. Once the triage is ruled and
written down, the build itself is Sonnet 5 (`claude-sonnet-5`) work.

## Phase 1 — Scope + prior art

1. Name the exact skill file(s) under `~/.claude/commands/` in scope, and read each
   one **end to end** before deciding anything. The skip rules, predicates and
   transforms buried in their prose are the spec — H556's own warning: *"a wrong
   predicate spams PRs to repos that should be skipped."*
2. `/prior-art` before writing a line: does a script already do this?
   - [`Uprava/tools/`](https://github.com/gasyoun/Uprava/tree/main/tools) is the tool
      home (665 Python scripts as of 03-09-2026) — grep it for the verb before adding one more.
   - The owning repo's own `tools/` / `scripts/` / `src/`.
   - [`SHARED_CODE.md`](https://github.com/gasyoun/github-spine/blob/main/SHARED_CODE.md)
     for a canonical helper in this family.
   H929 named this explicitly and it is the most-skipped step: `mdx_coverage.py`
   existed as precedent for the docx extraction and nearly got rebuilt.
3. Count the real cost before proposing the port. A skill that fans out across ~85
   repos to compute file-existence is an obvious win; a skill invoked twice a year
   over three items is not — say so and stop.

## Phase 2 — Triage: mechanical vs judgment (the core call)

Go clause by clause through each skill body and sort every step into exactly one bucket:

- **Mechanical → folds into the script.** Set-differences (which repos lack the file),
  file placement, templated PR bodies, `gh api` calls, counting, parsing, byte
  transforms, idempotent re-runs. Anything a regex or a dict lookup settles.
- **Judgment → stays LLM-gated, explicitly NOT folded.** Anything needing language
  understanding or a call a human would want reviewable. H556's canonical exclusions:
  [`/cologne-contributing-fix`](https://github.com/gasyoun/claude-config/blob/main/commands/cologne-contributing-fix.md)
  (is this existing CONTRIBUTING substantive or boilerplate?) and
  [`/cologne-tooling-runbook`](https://github.com/gasyoun/claude-config/blob/main/commands/cologne-tooling-runbook.md)
  were left alone **on purpose** — a fold there would have been a regression, not a win.

**Write the triage down in the handoff/PR body before building**, one line per clause
with its bucket. A skill that comes out ~100% mechanical is the H835/H929 shape (full
extraction, the skill becomes a thin caller). A skill that comes out mixed keeps its
`.md` as the judgment wrapper that *calls* the script for the mechanical middle. A skill
that comes out mostly judgment **should not be ported** — report that and stop; that is
a real, expected outcome of this phase, not a failure.

## Phase 3 — Design: config table + dynamic discovery

- **A config table, not a chain of `if`s.** H556's shape: each item maps to
  `(canonical_source, dest_path, applies_if_predicate, action)`. Adding the eighth
  deploy must be a table row, not a new code path.
- **Discover the item set dynamically — never a hardcoded list.** The seven
  `.deploy_*.py` scripts `cologne_batch_deploy.py` replaced each carried a hardcoded
  `ALL_REPOS` that had silently gone stale (SanskritLexicography was missing from the
  CodeQL list). Enumerate from the filesystem, the API, or the registry at run time.
- **Port every skip rule faithfully.** Each skill's "skip repos this doesn't apply to"
  logic becomes an `applies_if` predicate. Read all of them before writing any.
- **Preserve every transform verbatim** on an extraction (H929): the `make_mdx_safe`
  pass, the grid-wrapping, the encoding quirks. An extraction is a *move*, not a
  rewrite — resist improving the logic in the same pass, or the equivalence test in
  Phase 5 has nothing to prove.

## Phase 4 — Build

Land the script in `Uprava/tools/<name>.py` (the tool home — **not** `claude-config`,
whose `.gitignore` is an inverted whitelist that swallows new files silently), or the
owning repo's own `tools/` when the script is repo-local.

- **`--dry-run` is the default.** A bare invocation reports `present / missing /
  skipped(reason)` per item and writes nothing.
- **`--apply` is explicit and idempotent.** A re-run over an already-applied item is a
  no-op: zero PRs, zero API writes.
- **Scope flags** (`--deploy <name>`, `--repo <name>`, or the family's equivalent) so a
  single item can be exercised in isolation before the full sweep.
- **stdlib-first.** Reuse existing parsers rather than writing a third
  (`fruit_scan.py`, `build_dashboard_data.py`, `handoff_claims.py` all already parse
  the registries).
- **Windows/encoding:** `sys.stdout/stderr.reconfigure(encoding='utf-8')` and
  `encoding='utf-8'` on every output-capturing `subprocess.run(["gh", …])`; no UTF-8
  BOM. Both hook-enforced on full-file `.py` writes.
- Land the config table + `--dry-run` **first** and eyeball the output before `--apply`
  exists at all.

## Phase 5 — Prove equivalence (never skip)

The port is only done when old and new provably agree:

- **Extraction (H929):** run the OLD inline path and the NEW tool over the same real
  input and **diff the outputs — must be byte-identical.** Pick a genuine artifact from
  the repo, not a toy fixture.
- **Fan-out (H556):** hand-verify the `--dry-run` set-diff for **3 spot items** — one
  that has the file, one that lacks it, one that must be skipped. Then `--apply` to one
  test item: exactly one PR, correct path, CI green; re-run the identical command →
  zero PRs.
- **Guard/hook (H930):** reproduce the failure first, apply the fix, confirm it no
  longer reproduces **and that a control case still fires** — so the guard was fixed,
  not disabled.

If the outputs differ and the new one is *better*, that is still a failed equivalence
test: revert the improvement, land the faithful port, and file the improvement
separately. Otherwise nothing pins what the skill used to do.

## Phase 6 — Rewire the skill `.md` (the step that makes it real)

**A port is invisible until the skill file calls it.** Every skill in scope gets its
mechanical body replaced by an invocation of the script, keeping only the judgment
residue and a human spot-check note. A script that exists while the `.md` still tells
the agent to do the work by hand has changed nothing and cost tokens.

- Fully-mechanical skill → thin caller: trigger block, the one-line invocation, the
  acceptance note.
- Mixed skill → the `.md` keeps its judgment phases and calls the script for the middle.
- Consolidation → **each** folded skill's `.md` gets the one-line rewrite pointing at
  the shared script, and each keeps its own triggers (they stay separately invocable).
- Verify each rewired skill still triggers and its description still matches what it
  now does.

## Phase 7 — Register + sync

1. Register the script in the owning repo's `CLAUDE.md` Tools section (Uprava's is the
   canonical example) and, if it is a reusable helper,
   [`SHARED_CODE.md`](https://github.com/gasyoun/github-spine/blob/main/SHARED_CODE.md).
2. **G29** — Codex twin for any new/renamed skill, per
   [/claude-codex-skill-porter](https://github.com/gasyoun/claude-config/blob/main/commands/claude-codex-skill-porter.md).
3. **G34** — card in
   [Uprava/SKILLS_MANUAL_RU.md](https://github.com/gasyoun/Uprava/blob/main/SKILLS_MANUAL_RU.md)
   (via a worktree — Uprava blocks main-tree commits).
4. **G35** — row in
   [SKILLS_INDEX.md](https://github.com/gasyoun/github-spine/blob/main/SKILLS_INDEX.md);
   a consolidation must fix the count, not just add a row.
5. `CHANGELOG.md` `[Unreleased]` entry + `/cut-release` per the standing cadence, in
   each touched repo. `/handoff-close` if this run executed a named handoff.

## Mode — cross-harness residue-port (`--residue-port`, U59)

The nine-session port series of 04-09-2026 ([H4096](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H4096-OxAlpha_Uprava_zj-findings-append-port_04.09.26.md)–[H4104](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H4104-OxAlpha_Uprava_zj-org-axis-sweep-port_04.09.26.md)) ran a sibling shape: not consolidating one fan-out, but moving ONE skill's **zero-judgment residue** into a committed `tools/*.py` of the **CONSUMING repo**, so opencode/codex/grok execute the mechanics without a Claude command. Approved **U59** (skill-mine delta census H4060–H4304, MG chat vote 5/5 approve 07-09-2026).

- Triage the skill's clauses: everything deterministic (append a findings row, emit a metadoc skeleton, compute a table + `--check`, sweep a convention) is portable residue; judgment stays in the skill and says "run `python <repo>/tools/<name>.py`".
- The tool lives in the consuming repo's `tools/` with an H2081-style write marker + `writer_contracts.json` row when it writes tracked files; selftest with positive AND negative; `--dry-run` default where it mutates.
- Wiring contract: the skill `.md` (and its Codex/Grok twins) names the tool and its flags; the tool's `--selftest` is the verification entry; changelog row in the consuming repo.
- Variants seen in the series: `collision-grep` first (H4103), `--check` parity gate (H4099), `wire-existing-tool` instead of authoring (H4098), report-only verdict (H4102), harness-shape for census fans (H4104).

## Gotchas

- **The triage is the deliverable.** Folding a judgment step produces a script that is
  confidently wrong at scale — worse than the fan-out it replaced.
- **A hardcoded list is the defect you are fixing.** If the new script hardcodes what
  the old skill discovered, the port went backwards.
- **Never `--apply` before a human has read a `--dry-run`.** These scripts open PRs
  across ~85 repos; a bad predicate is an org-wide spam event.
- **Auth scope bites late:** branch-protection/ruleset writes need `administration`
  scope. Dry-run needs only read, so the failure surfaces at `--apply`. Confirm first.
- **Don't touch `claude-config` root** — its `.gitignore` opens `/*` and then re-admits
  each allowed path by hand. `/commands/`, `/hooks/`, `/rules/`, `/agents/`, `/skills/`,
  `/references/` and `/tools/` are whitelisted **as directories**, but `/scripts/` is
  re-excluded (`/scripts/*`) and every single script is re-admitted by name
  (`!/scripts/<file>.py`, ~68 of them). So a new file under `scripts/` is invisible to
  `git add` until its own `!` line exists — check with
  `git check-ignore -v scripts/<new>.py` **before** committing, or the work is silently
  dropped (this is how H3956's script was lost). Tools still belong in `Uprava/tools/`.
- Uprava takes direct pushes to `main`, but **only from a worktree** — the shared
  main-tree commit guard blocks the rest.

## When this isn't the right tool

- A hand-maintained **table/count** drifting from its source → that's
  [/computed-table](https://github.com/gasyoun/claude-config/blob/main/commands/computed-table.md)'s
  `--emit`/`--check` parity gate, not a skill port.
- Syncing a skill between Claude and Codex → `/claude-codex-skill-porter`.
- Finding *which* skills should exist at all → `/skill-mine`.
- A skill whose work is genuinely judgment (referee passes, register-sensitive
  rewriting, editorial calls) — Phase 2 will say so; the correct output is "don't port
  this", reported and stopped.

Evidence: H556 · H835 · H929 · H930 (tiered above, all in Uprava handoffs archive).

_Dr. Mārcis Gasūns_
