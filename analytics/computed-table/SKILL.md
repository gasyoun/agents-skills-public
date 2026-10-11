---
name: "computed-table"
description: 'Turn a hand-maintained table/count in a doc into a derive-don''t-store --emit/--check parity gate - identify the computable source, write a tools/name_check.py generator, wire it as hook/CI, prove idempotence. Triggers: "computed table", "derive-don''t-store".'
---

# computed-table

Use this skill when the user asks to run the migrated source command `computed-table`.

## Command Template


# /computed-table

_Created: 11-07-2026 · Last updated: 05-09-2026_

A hand-maintained table or count in a doc **will** drift from the reality it describes — not
occasionally, within 48 hours. H526 (`goals_check.py`) proved the fix: stop hand-editing the
table, generate it from a machine-readable source, and gate the parity so drift can't land
silently. This skill is the recipe, extracted after the fourth independent repetition in a
week (H526 → H555 → H609 → H615) — build the fifth one from the template below instead of
reinventing the shape.

**Argument:** the target — a doc + the table/count that's drifted (e.g. `ROADMAP_INDEX.md
Forks column`, `SKILLS_INDEX.md skill count`), or a clear reference to one discussed in the
conversation.

## Default path (seconds — run this first)

From the Uprava clone root, against an explicit source register and target doc:

```sh
python tools/computed_table.py --source <register> --target <doc.md>           # dry-run report
python tools/computed_table.py --source <register> --target <doc.md> --apply   # emit/refresh the block
python tools/computed_table.py --source <register> --target <doc.md> --check   # parity gate (exit 0 = current)
```

[`tools/computed_table.py`](https://github.com/gasyoun/Uprava/blob/main/tools/computed_table.py)
(H4099 — the mechanical extract) parses the register you NAME (JSON list /
`{"columns","rows"}` / JSONL / markdown pipe table / TSV; `#`-comments skipped;
`--header A,B,C` for schema-in-comment registers), applies the filter/order
you pass (`--where col=val`, `--sort`/`--desc`, `--columns`, `--count`,
`--sum`), and splices the rendered GFM table between
`<!-- computed-table:BEGIN/END -->` markers — dry-run by default, idempotent
on re-apply, hand prose outside the markers never touched. LLM / judgment
residue only after the script report exists: which source register is
canonical for the ask, and the per-column aggregation intent, stay judgment
calls — the tool never guesses a register. Phases 2–5 below remain the
recipe for bespoke per-target generators (`tools/<name>_check.py`) wired
into hooks/CI/goals.

## The two failure modes this kills

1. **Silent count drift** — a header says "N rows" or "M skills"; the table itself has
   N±k. Nobody notices until an unrelated audit trips over it (H609's `SKILLS_INDEX.md`
   found "119 skills" claimed against 120 actual rows at the time — a historical snapshot,
   not the live count, which has since grown well past that).
2. **Fictional status claims** — a doc line describes an enforcer, a hook, a gate that
   never existed or was silently removed — because the line was typed once and never
   re-verified against the thing it claims (H526's goals docs had exactly one of these
   within 48h of hand maintenance).

Both are the same root cause: a claim about disk state, re-typed by hand instead of
re-derived from disk state.

## Phase 0 — qualify (is this worth automating?)

Not every table earns a generator. The bar: the table/count is **re-derivable from
something already on disk** (files, rows in another table, git log, a JSON crosswalk) AND
it has **already drifted at least once**, or the doc is edited often enough that drift is a
matter of when, not if. A one-time snapshot table that nobody updates isn't a candidate —
leave it hand-authored. If the subject clearly fails the bar, say so in one line and stop.

## Phase 1 — identify the source and the generated block

1. **The computable source** — what does the table's truth actually derive from? Usually
   one of: rows on disk (a glob over files matching a naming convention), another table
   already in the repo, a small hand-authored JSON/YAML crosswalk (when the data itself is
   judgment, e.g. `goals_matrix_data.json`), or `git log`/file metadata.
2. **The generated block** — the exact span of the doc that becomes machine-written. Keep
   it as narrow as possible: a marker-fenced block (`## Matrix — …` through the next `##`
   heading is the house convention, see `goals_check.py`'s `render_block`), or in-file
   digit-only substitutions (see `registry_check.py`'s header-count/marker rewrites). Hand
   prose — framing, judgment notes, caveats — stays OUTSIDE the generated span and is never
   touched by `--emit`.

## Phase 2 — write `tools/<name>_check.py`

Two modes, same script, same shape as every house precedent
([`goals_check.py`](https://github.com/gasyoun/Uprava/blob/main/tools/goals_check.py),
[`registry_check.py`](https://github.com/gasyoun/Uprava/blob/main/tools/registry_check.py),
[`roadmap_index_check.py`](https://github.com/gasyoun/Uprava/blob/main/tools/roadmap_index_check.py),
[`skills_manual_check.py`](https://github.com/gasyoun/Uprava/blob/main/tools/skills_manual_check.py),
[`skills_index_check.py`](https://github.com/gasyoun/Uprava/blob/main/tools/skills_index_check.py)):

- **`--emit`** — rescan the source, render the block fresh, splice it back between the
  markers (or rewrite only the digit spans for a header-count style target). Byte-surgical:
  only the generated span changes, never surrounding hand prose. Print what changed (or
  "already current" — a no-op emit is not an error).
- **`--check`** (default when no flag given) — render fresh in memory, diff against what's
  actually in the file, exit 0 if identical, exit 1 with a `FAIL:` line naming exactly what's
  stale ("run `--emit`") otherwise. This is the parity gate.
- **`--ci`** (only if the check needs sibling repos NOT guaranteed present in a GitHub
  Actions runner) — a repo-local subset that's CI-safe; the full check stays a local/
  `/weekly-review` pass. An org-wide recursive glob over `GitHub/` is a standing footgun —
  it times out. Scope every glob to a named repo or a bounded subtree.
- **Repo-alias pattern** — if the target resolves sibling-repo paths, reuse `goals_check.py`'s
  `REPO_ALIASES` / `registry_check.py`'s `repo_root()` idea: pin "this repo" to the checkout
  the script is actually running from (so the gate stays self-consistent inside a worktree),
  and treat repos not locally cloned as SKIPPED, never FAILED.

## Phase 3 — replace the hand block, add the banner

The generated span gets the standard banner immediately above it (verbatim pattern from
`goals_check.py`'s `render_block`):

```markdown
> **Generated section — regenerate, never hand-edit.** … emitted by
> [`tools/<name>_check.py`](<full blob URL>) `--emit` from <the source>.
> `--check` gates parity<, wired as G### if this repo has a goals system>.
```

## Phase 4 — wire the gate

Pick one, in order of preference:

1. **Pre-commit hook** (`.githooks/` + `git config core.hooksPath .githooks`) — catches
   drift before it ever lands, cheapest to run locally.
2. **CI step** running `--check` (or `--ci` for the repo-local subset) — for repos with
   GitHub Actions already wired; only viable if the check doesn't need sibling repos.
3. **Weekly-review / `/goal`-loop check** — for org-wide checks that genuinely need
   multiple repos checked out locally (`roadmap_index_check.py`'s case); not CI-safe, stays
   a periodic local pass instead. Say explicitly why CI wasn't chosen — don't leave the gate
   unwired without a reason on record.

If the repo runs the Uprava goals system, register a 🔁 goal row for the new gate via the
five-step minting checklist in
[`GOALS_MANUAL.md`](https://github.com/gasyoun/Uprava/blob/main/GOALS_MANUAL.md) (next free
G# + JSON row in `goals_matrix_data.json` + `goal:` frontmatter on any skill wrapper +
`goals_check.py --check` green in the same commit) — this is what turns a one-off script
into something `/weekly-review` and the coverage matrix actually track.

## Phase 5 — prove idempotence

Run `--emit` twice. The second run's diff must be empty — a generator whose own output
isn't a fixed point will fight hand edits or itself. This is a required acceptance check,
not optional polish.

## Acceptance

- `tools/<name>_check.py` committed, `--check` exits 0 against the freshly-`--emit`'d file.
- Second `--emit` run is a no-op (idempotence proven).
- The gate is wired (hook, CI, or an explicit "why not CI" note) — an unwired generator is
  just a script nobody runs.
- The generated banner names the source of truth and points at the emitting script.

## Guardrails

- **Additive tooling only** — don't refactor the doc's hand-authored prose while you're in
  there; scope creep on a mechanical build muddies the parity-gate diff.
- **Hand judgment never gets auto-fixed.** If a census surfaces something that needs a human
  call (H609's `skills_manual_check.py` found 3 uncarded skills — it WARNed, it didn't
  fabricate cards), report it, don't synthesize.
- **Narrow generated span.** The wider the block `--emit` owns, the more likely it clobbers
  something a human meant to hand-edit next to it.
- **Shared-tree repos (Uprava, SanskritLexicography) under concurrent minting** — commit via
  an isolated worktree off `origin/main` with push-retry, never `git add -A`.

## Related

[`GOALS_MANUAL.md`](https://github.com/gasyoun/Uprava/blob/main/GOALS_MANUAL.md) (the goal
system this pattern feeds), `/metadoc` (same "generated vs hand-authored" split, for prose
instead of tables), `/md-hygiene`, `/handoff-registry-audit` (a `registry_check.py`
consumer), the four house precedents linked in Phase 2.

_Dr. Mārcis Gasūns_
