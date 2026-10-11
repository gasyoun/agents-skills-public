---
name: "link-audit-fix"
description: 'Org-wide broken-link audit + auto-fix fan-out across repos under GitHub/ — resolves relative, #anchor, blob/tree links vs local clones, only re-verified fixes, hard default-branch guard. Triggers: "link audit", "fix broken links org-wide".'

---

# link-audit-fix

Use this skill when the user asks to run the migrated source command `link-audit-fix`.

## Command Template


# /link-audit-fix

_Created: 17-07-2026 · Last updated: 26-08-2026_

No argument = audit every repo under `GitHub/`; a repo name scopes to one.

Born from a real recurring pattern, not a hypothetical: an "org-wide link audit,
auto-fix pass" was run by hand on 14-07-2026 (commit message pattern `fix: repair
N of M broken link(s) (org-wide link audit, auto-fix pass)`, 60+ commits across
40+ repos) with no committed tool behind it — and it had a serious defect
([FINDINGS.md §84](https://github.com/gasyoun/Uprava/blob/main/FINDINGS.md)):
it committed each repo's fixes to *whatever branch happened to be checked out*
in the shared main tree, stranding real fixes on ancient feature branches
(`h540-typed-link-lint-fix`, `docs/metadoc-template-v2-backfill-h663`) instead
of landing on `main`/`master`. This skill is that procedure, encoded once,
with the branch-safety bug fixed at the tool level so it cannot recur.

**Tier:** mechanical scan + judgment-gated fix proposals — Sonnet 5
(`claude-sonnet-5`); escalate only if a repo's link corpus needs a genuinely
novel resolution heuristic beyond Phase 2's three.

## Phase 0 — Dedupe gate (CENSUS GATE — do this before touching anything else)

This candidate looked, on the surface, like it might already be owned by two
existing skills. It is not — but confirm the boundary before building:

- [`/md-hygiene`](https://github.com/gasyoun/claude-config/blob/main/commands/md-hygiene.md)
  fixes link **form** (relative → full blob URL, dated headers, HTML-in-md) —
  it never checks whether a link's *target* actually exists.
- [`/crosslink-weave`](https://github.com/gasyoun/claude-config/blob/main/commands/crosslink-weave.md)
  + [`tools/crosslink_weave_check.py`](https://github.com/gasyoun/Uprava/blob/main/tools/crosslink_weave_check.py)
  audit link **density within one hub doc** (enough internal + evidence links
  per section) — it flags dead in-doc anchors as a side effect, but never
  scans cross-repo or resolves a github.com blob URL against another clone.
- Neither does org-wide broken-link **detection + repair**. `FINDINGS.md §84`
  is the proof this gap is real and already cost real broken commits — verify
  that finding is still current (`grep -n "§84" Uprava/FINDINGS.md`) before
  assuming the gap persists.

If a later sweep shows either skill absorbed this scope, downgrade this skill
to an UPDATE note pointing at the absorbing one — don't maintain three tools
for one job.

## Phase 1 — Run the audit script (dry-run by default)

```sh
# from the Uprava clone root (canonical script: tools/link_audit_fix.py)
python tools/link_audit_fix.py                    # all repos, report only
python tools/link_audit_fix.py --repo NAME        # scope to one repo
python tools/link_audit_fix.py --fetch            # fresher github.com resolution
python tools/link_audit_fix.py --json out.json    # machine-readable dump
```

[`tools/link_audit_fix.py`](https://github.com/gasyoun/Uprava/blob/main/tools/link_audit_fix.py)
does the zero-judgment scan + resolution + fix-proposal work; read its module
docstring before touching it, and reuse its `github_slug`/anchor logic (ported
from `crosslink_weave_check.py`, not reinvented) rather than writing a second
copy. **Run the dry-run pass against each repo's normal sibling-of-`GitHub/`
checkout, never against an isolated worktree** — relative cross-repo mentions
(`x` written from the org root) only resolve correctly
when sibling repos are present on disk next to the scanned repo; an isolated
worktree copy has no such siblings and will report false positives for every
cross-repo relative mention. Dry-run is read-only and safe to run directly
against a dirty/shared main tree.

**Performance note — FIXED 04-08-2026, but know the history.** The script used to
spawn a `git` subprocess per `github.com` link resolved (in fact *two*: a repeated
`remote get-url` plus `cat-file -e`), and Windows process-spawn overhead — not git —
dominated everything. On Uprava after the H1798 registry split that meant **15,326
blob/tree links across 1,036 files**, i.e. ~31k spawns: two scans were killed at
35 min and 2 h having printed nothing at all, because the report is only emitted at
the end (there is no progress signal — an apparently-hung run may be working fine).

It now resolves one `ls-tree -r` per (clone, ref) into a membership set, with the
remote-org, fetch and anchor lookups memoized: the same Uprava scan takes **~4 min**.
Scoping with `--repo NAME` is still the right default for iteration; a full org sweep
is now practical rather than a last resort.

The scan resolves, entirely locally (no network unless `--fetch`):

1. **Relative markdown links** (`./x.md`, `../x.md`, `x.md`) — resolved on
   disk relative to the source file.
2. **`#anchor` links** (same-file or `file.md#anchor`) — resolved against the
   target file's GitHub heading-slug set.
3. **Full `https://github.com/<org>/<repo>/(blob|tree)/<branch>/<path>` URLs**
   — resolved against a local clone of `<repo>` under `GitHub/` via
   `git cat-file -e <branch>:<path>` (no checkout needed, so this never
   touches the target repo's working tree).

`issues/`, `pull/`, `commit/`, `releases/` links and non-github external URLs
are counted but never verified (would need network) — out of scope by design,
not a gap to close here.

## Phase 2 — Fix proposals are guesses until re-verified — never apply blind

For every broken link the script proposes at most one fix, from three
heuristics, and **only after re-running the exact same resolution check
against the proposed replacement**:

1. `handoffs/<f>.md` ↔ `handoffs/archive/<f>.md` — the single most common
   archive-lag drift the org's own commit history shows.
2. `master` ↔ `main` branch swap on a `github.com` blob/tree URL, when the
   other branch resolves.
3. A single unambiguous basename match elsewhere in the same repo tree
   (`git ls-files` filtered by basename) — zero or 2+ matches propose
   nothing; ambiguity is reported, never guessed.

This is not decorative caution: the 14-07 hand-run pass proposed 25 fixes and
**2 were wrong** — `gasyoun/MWS` and `sanskrit-lexicon/SanskritGrammar`, neither
of which exists (real orgs: `gasyoun/SanskritLexicography`... see
[reference_repo_org_ownership] convention: SanskritLexicography + VisualDCS
are `gasyoun` personal, not `sanskrit-lexicon`). Both were caught only because
someone manually re-checked before applying. This tool's re-verification step
makes that check automatic and mandatory — a fix that doesn't re-resolve is
never proposed as fixable at all, so there is no "trust me" step for a human
to skip.

A repo with broken links and no safe fix is not a failure of this skill —
report it as "no safe fix found, needs a human/judgment call" and move on.
Never invent a target.

## Phase 3 — Apply, with the branch-safety gate `FINDINGS.md §84` demands

```sh
# unguarded repo, on its default branch
python tools/link_audit_fix.py --apply --repo NAME

# GUARDED repo (Uprava, SanskritLexicography, csl-atlas, +13) — from a worktree
python /Users/<you>/Documents/GitHub/<Repo>-<slug>/tools/link_audit_fix.py \
    --apply --allow-worktree --repo <Repo>-<slug>
```

Before committing anything in a repo, the script checks:

- the repo's **checked-out branch name equals its own actual default branch**
  (`origin/HEAD` symbolic-ref, or whichever of `main`/`master` exists as a
  remote-tracking branch) — if not, the repo is **skipped with a loud reason**,
  never committed to whatever happens to be checked out;
- the working tree is otherwise clean before this run touches it.

This is the exact gap that stranded the 14-07 pass's fixes on
`h540-typed-link-lint-fix` and similar branches in `kosha` and
`SanskritLexicography` shared main trees.

### `--allow-worktree` — required on the 16 guarded repos

**On a guarded repo the branch-equality check above is unsatisfiable**, and for a
while that made `--apply` simply unusable there: the gate demands you be *on* the
default branch, while the repo's `.githooks/pre-commit` refuses any commit made in
the main-tree checkout — the only place that can hold. The two protections exclude
each other. This was found the hard way on 04-08-2026, when a Uprava pass had to
apply 469 verified fixes by hand because `--apply` could not run at all.

`--allow-worktree` waives **only** the branch-equality check, and **only** inside a
linked worktree (`.git` is a file, not a directory) — which is by construction this
session's own tree on its own branch, so the someone-else's-branch hazard §84 exists
to prevent is absent. It is not a general bypass:

- passing it in a **primary checkout is refused outright**, with a loud reason;
- the **clean-tree** check is untouched and still refuses a dirty worktree;
- omitting it leaves behaviour byte-identical to before, except the refusal message
  now names the flag instead of dead-ending.

Pinned by [`tools/test_link_audit_worktree_optin.py`](https://github.com/gasyoun/Uprava/blob/main/tools/test_link_audit_worktree_optin.py)
(16 checks over a real clone + a real linked worktree, verified RED against a broken
detector). Run the worktree's **own** `tools/` copy, per the Uprava tree-guard rule.

The script **commits but never pushes** — push or PR per the repo's own convention
(direct push for a private single-owner repo, PR for a shared/public one;
`csl-orig` is never committed to directly by agents at all, per the org
CLAUDE.md correction workflow).

### Never auto-merge a repo we do not own — test OWNERSHIP, not the name

A sweep touches many repos at once, which is exactly when this goes wrong. Before
merging anything, ask **"is this repo owned outside `gasyoun` / `sanskrit-lexicon`?"**
If yes, the PR **stays open for its maintainer** — it is their repository, however
benign and tool-verified the diff is. **An owner you cannot determine counts as
external, not as ours.**

Do **not** carry a hardcoded list of excluded repo names into the sweep. On
04-08-2026 exactly that mistake auto-merged
[drdhaval2785/SanskritSpellCheck#73](https://github.com/drdhaval2785/SanskritSpellCheck/pull/73)
into an external maintainer's own `master` (reverted by
[#74](https://github.com/drdhaval2785/SanskritSpellCheck/pull/74)): the exclusion had
been copied as the names `CORRECTIONS` / `csl-devanagari` / `csl-orig`, and that repo
matched none of them. A name list only ever excludes repos someone thought of in
advance. Enforced by
[`hooks/remind_pr_automerge.py`](https://github.com/gasyoun/claude-config/blob/main/hooks/remind_pr_automerge.py)
(pinned by [`hooks/test_remind_pr_automerge.py`](https://github.com/gasyoun/claude-config/blob/main/hooks/test_remind_pr_automerge.py)),
but the hook only *warns* — in a scripted fan-out that bypasses `gh pr create`, the
check is yours to make.

## Phase 4 — Re-land safe passes from the 14-07 stranding

Any commit still sitting on a stale non-default branch matching the message
pattern `fix: repair N of M broken link(s) (org-wide link audit, auto-fix
pass)` is stranded, unlanded work (`FINDINGS.md §84`'s own remediation note).
Per-file triage before cherry-picking — some of those fixes are already
superseded by independent edits to the same lines on the real default branch.
Route through `/branch-contention-recover` if a repo's stranded branch has
since diverged further.

## Phase 5 — Report

One table per repo with findings: file, broken target, detail, proposed fix
(or "none — needs human"), applied y/n. Totals: repos swept, links broken,
fixes proposed, fixes applied+committed, repos skipped by the branch-safety
gate (name each, with the branch it was stuck on). A clean sweep reports
**"0 broken links"** explicitly across however many repos were scoped — silence
is not a result.

## Guardrails

- Never fabricate a replacement URL/path — every proposed fix is independently
  re-verified before it is even offered as an `--apply` candidate.
- Never commit to a repo whose checked-out branch isn't its own default branch
  — this is not a suggestion, the script enforces it and skips otherwise. The
  one sanctioned exception is `--allow-worktree` inside a **linked worktree**,
  which the script verifies for itself and refuses in a primary checkout.
- A shared/concurrently-minted tree (Uprava, SanskritLexicography, and the
  other guarded repos) means `--apply --allow-worktree` runs in a worktree,
  never the dirty main checkout — `git add -A` is never appropriate here
  either; the script only ever stages the exact files it edited.
- **Put that worktree under `GitHub/` as a sibling** (`GitHub/<Repo>-<slug>`),
  not somewhere isolated: relative cross-repo mentions only resolve when the
  sibling repos sit next to the scanned tree, so an isolated worktree turns
  every one of them into a false positive (Phase 1's warning applies to the
  apply pass too).
- Dry-run (Phase 1) is safe anywhere, including a dirty shared tree, since it
  never writes. Only Phase 3 (`--apply`) needs the worktree isolation.

## Related

[`tools/link_audit_fix.py`](https://github.com/gasyoun/Uprava/blob/main/tools/link_audit_fix.py)
(this skill's companion script) · `/md-hygiene` (link *form*, not resolution)
· `/crosslink-weave` + `tools/crosslink_weave_check.py` (link *topology* within
one doc, and the source of the `github_slug` anchor algorithm reused here) ·
`/branch-contention-recover` (routes a stranded-branch re-land) ·
`FINDINGS.md §84` (the branch-stranding gotcha this skill's Phase 3 gate
exists to make structurally impossible) · H1131 (Sonnet 5) — claude-config skill
build link-audit-fix, census item #24 of the [H1102 (Fable 5) — all-executor skill census (30 skills)](https://github.com/gasyoun/Uprava/blob/main/archive/SKILL_CANDIDATES_CENSUS_ALL_EXECUTORS_12_17_07_2026.md)).

## When this isn't the right tool

- Fixing a link's **form** (relative → full blob URL) when the target is
  already correct — that's `/md-hygiene`.
- Auditing whether one hub doc's sections have enough internal/evidence
  links — that's `/crosslink-weave`.
- Checking external (non-github.com) URLs for link rot — out of scope; that
  needs real network requests (see `RussianRamayana/scripts/check_links.py`
  for a single-repo precedent of that different problem, not reused here
  since the domain — external HTTP link rot vs. internal cross-repo markdown
  resolution — doesn't overlap).

_Dr. Mārcis Gasūns_
