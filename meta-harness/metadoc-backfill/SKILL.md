---
name: "metadoc-backfill"
description: 'Batch-backfill genre-named docs lacking a sibling stem.meta.md across a repo/org — reconcile Uprava/METADOCS_INDEX.md, generate v2-template metadocs, commit; batch sibling of /metadoc. Sonnet 5 (claude-sonnet-5) tier. Trigger: "backfill the metadocs".'
---

# metadoc-backfill

Use this skill when the user asks to run the migrated source command `metadoc-backfill`.

## Command Template


# /metadoc-backfill

_Created: 14-07-2026 · Last updated: 29-08-2026_

## Default path (seconds — run this first)

From the Uprava clone root:

```sh
python tools/metadoc_backfill_scan.py --all
```

LLM / judgment residue only after the script report exists (or the script exits non-zero with a real defect). Qualify-and-fill stays after that report.

The authorship-time metadoc-coverage hook ([`stop_metadoc_coverage_check.py`](https://github.com/gasyoun/claude-config/blob/main/hooks/stop_metadoc_coverage_check.py)) fires **once per session, warn-only, on whatever genre-named doc happened to be dirty** — it never sweeps a repo, never touches the org census, and never creates anything. [`/metadoc`](https://github.com/gasyoun/claude-config/blob/main/commands/metadoc.md) closes exactly **one** gap on request. Nothing walks a whole tree and closes *every* missing metadoc in one pass. This skill is that pass — the repeatable form of the
[H663](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H663-Sonnet_claude-config_metadoc-template-v2-backfill_11.07.26.md)
backfill (Sonnet 5 — metadoc template v2 + backfill).

**Argument** (`$ARGUMENTS`): a repo name (`SanskritGrammar`) scopes it to that clone; `all` (or empty → prompt, default `all`) sweeps every repo under your GitHub clones directory.

This is judgment work, not a rubber stamp — each candidate must clear the Phase 0 bar before it earns a metadoc, and each stub must be *filled* from the subject, not templated blank. Sonnet 5 (`claude-sonnet-5`) tier.

## Phase 0 — scope + the genre classifier (reuse, don't reinvent)

The definition of "genre-named doc that wants a metadoc" is **already encoded** in the hook's `is_genre_doc()` + `GENRE` / `EXEMPT_EXACT` sets ([`stop_metadoc_coverage_check.py`](https://github.com/gasyoun/claude-config/blob/main/hooks/stop_metadoc_coverage_check.py) lines 53–92). Do NOT hand-roll a competing regex — port that exact logic:

- **Candidate** = a `*.md` whose stem contains any `GENRE` token (`roadmap plan spec memo report review audit summary analysis proposal outline agenda findings notes draft …`) OR is a manual / hub-README-of-record.
- **Not a candidate** = `EXEMPT_EXACT` (`README.md` unless it's a README-of-record, `CLAUDE.md`, `CHANGELOG.md`, `.ai_state.md`, `FINDINGS.md`, hub registries), `H###-…` handoffs, anything already ending `.meta.md`, and anything under an `archive/` path segment.
- **Missing** = no sibling `<stem>.meta.md` next to it.

Run the committed scanner — [`Uprava/tools/metadoc_backfill_scan.py`](https://github.com/gasyoun/Uprava/blob/main/tools/metadoc_backfill_scan.py)
(H2791 — Sonnet 5, the metadoc backfill scanner) — instead of writing a throwaway `.py` each session:
`python tools/metadoc_backfill_scan.py --repo-name <repo> <path>` from the Uprava clone root for a single clone, or `--all` for a whole-org sweep (it already scopes to subdirectories and skips the org-root timeout). It walks the repo(s) with `git ls-files '*.md'` (tracked only — untracked scratch doesn't earn a metadoc), applies the exact ported `is_genre_doc()`/`GENRE`/`EXEMPT_EXACT` classifier, and emits the `(repo, path)` list with no sibling metadoc. Pass `--reconcile` to cross-check straight against [`METADOCS_INDEX.md`](https://github.com/gasyoun/Uprava/blob/main/METADOCS_INDEX.md) Coverage-gaps in the same run (folds into Phase 1 below). Verify the classifier is still current with `--selftest` before trusting a live run.

## Phase 1 — reconcile against the org census FIRST

Before generating anything, read [`Uprava/METADOCS_INDEX.md`](https://github.com/gasyoun/Uprava/blob/main/METADOCS_INDEX.md) — both the narrative census table and the computed section built by [`tools/metadocs_index_build.py`](https://github.com/gasyoun/Uprava/blob/main/tools/metadocs_index_build.py). Cross-check your Phase 0 list against it:

- A doc already rowed as covered but with **no sibling on disk** → a census defect (worktree/branch drift, like the Systema stale-branch case in the index header). Note it; don't double-create.
- A doc your scanner missed but the census lists → the census seed is broader than `git ls-files` on the current branch (check you're on `main`/`master`, not a stale feature branch — that undercounts).
- A candidate the census already flags under **"Coverage gaps"** → that's your work list; prioritise those.

The reconciled list = genuine missing metadocs. This is the "check prior art before building" reflex applied to the census.

## Phase 2 — qualify each candidate (the narrow bar, per-item)

Genre-named ≠ metadoc-worthy. For each reconciled candidate apply the [`/metadoc`](https://github.com/gasyoun/claude-config/blob/main/commands/metadoc.md) Phase 0 test: **is it long-lived, consulted across sessions, worth maintaining?** Reject and record a one-line reason for:

- throwaway / scratch / dated-snapshot notes that won't be reopened,
- auto-generated files (dashboards, computed censuses, `_Auto-generated by X_` outputs),
- a trivial one-file repo README already self-describing.

A batch that manufactures a metadoc for every genre-named disposable file is as useless as no metadocs. Keep the rejects in the run report as `skipped: <reason>` — don't silently drop them.

## Phase 3 — generate a FILLED v2-template stub per survivor

For each survivor, create the sibling `<stem>.meta.md` using the **v2 template from [`/metadoc`](https://github.com/gasyoun/claude-config/blob/main/commands/metadoc.md) Phase 2a** — same section skeleton, same order, in the **subject's language** (Russian subject → Russian metadoc):

`## Предмет` (Purpose/Audience/Format) · `## Провенанс` (Created date + this backfill handoff + tier+version) · `## Бэклог улучшений` (ranked) · `## Известные ограничения` · `## Intended use / known misuse` · `## Maintenance & sunset plan` · `## Deprecation status` (`active` default; `superseded by [X]` / `retired` for dated snapshots) · `## Связанные документы` · `## История ревизий`.

**Fill, don't stub-blank.** Read enough of the subject to write a real one-line purpose, name the real audience, and seed **at least one honest backlog row** (or an explicit `parked — <reason>` if genuinely none) — every open backlog row cites an owner (`H###`) or is marked `parked`; an ownerless "open" row is a defect. Provenance:

- Real creation date: `git log --follow --diff-filter=A --format=%ad --date=short -- <subject> | tail -1` → `DD-MM-YYYY` (untracked → today).
- Authored-MD contract (hook-enforced): dated header `_Created: … · Last updated: …_`, `_Dr. Mārcis Gasūns_` byline, **full `blob`/`tree` URLs** (never relative), plain Markdown (no HTML), `DD-MM-YYYY` dates, model attribution as **tier + exact version**.

At batch scale, if the whole backfill is itself the durable artifact, mint ONE handoff for the pass via [`/handoff-mint`](https://github.com/gasyoun/claude-config/blob/main/commands/handoff-mint.md) (`Uprava/tools/mint_handoff.py`) and cite that H### as the provenance in every stub — don't hand-pick the number.

## Phase 4 — regenerate the census (derive, don't hand-edit)

Run [`tools/metadocs_index_build.py`](https://github.com/gasyoun/Uprava/blob/main/tools/metadocs_index_build.py) to regenerate the computed census section — it is derive-don't-store; **do not hand-edit that block**. Then update the **narrative** census table above it: add a row per new metadoc, or — for a large sweep (a full-repo coverage pass) — **one batch row** naming the count, the repo, the handoff(s), and a release/PR link (the Systema `docs/` 60-metadoc batch row is the precedent). Bump `Last updated:` in the index header with a one-line what-changed note (count delta + tier+version).

## Phase 5 — commit

- **Per-repo**, in the repo that owns each subject: commit through a worktree off `origin/<default>` (the shared-main-tree guard blocks main-tree commits — see [`/land-on-main`](https://github.com/gasyoun/claude-config/blob/main/commands/land-on-main.md), which owns the guarded-main landing mechanics); public repos → PR with auto-merge, `Uprava` → direct push to `main`.
- The handoff minted in Phase 3 scopes the work → commit → PR → merge with **no confirmation ask** (the standing handoff-autonomy rule).
- CHANGELOG entry in each touched repo (`[Unreleased] → Added`: "N metadocs backfilled").
- Run the [`/document`](https://github.com/gasyoun/claude-config/blob/main/commands/document.md) sweep once for the pass so the census, GTD, and any sibling registries reflect the new coverage.

## Output

A run report: candidates scanned · reconciled-missing · **generated** (with paths) · **skipped** (each with its one-line reason) · census delta (before → after count) · handoff ID · commit/PR links per repo.

## When NOT to use

- **A single missing metadoc** → use [`/metadoc`](https://github.com/gasyoun/claude-config/blob/main/commands/metadoc.md); a batch pass for one doc is overhead.
- **Throwaway / auto-generated / dated-snapshot files** — these are the Phase 2 *rejects*, never force a metadoc onto them.
- **Judging the shape/quality of ONE existing metadoc** — that's [`/metadoc`](https://github.com/gasyoun/claude-config/blob/main/commands/metadoc.md) update mode, not a coverage sweep.
- **`csl-orig`** — agent-barred (the hook excludes it); never author into it.

## Safety notes

- **Reconcile before generate** (Phase 1) — creating a metadoc the census already tracks on another branch produces a duplicate; check `main`/`master` first.
- **Never hand-edit the computed census** — always regenerate via the build tool; hand-edits drift from reality on the next run.
- **Fill from the subject, don't blank-template** — a wall of empty-skeleton metadocs is the "🟠 all-open" rot the index warns about; each must carry a real purpose + ≥1 owned-or-parked backlog row.
- **Worktree, not main-tree** — the pre-commit guard blocks canonical-checkout commits; batch commits especially must land through worktrees to stay clear of concurrent sessions.

_Dr. Mārcis Gasūns_
