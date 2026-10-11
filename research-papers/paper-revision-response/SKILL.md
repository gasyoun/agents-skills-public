---
name: "paper-revision-response"
description: 'Apply an existing referee report''s Major/Minor findings via PR and generate point-by-point RESPONSE_TO_REVIEWERS_Axx.md, bumping readiness 3→4 in ARTICLES.md. Triggers: respond to the review, revision pass for Axx. Sonnet 5 (claude-sonnet-5) tier. Do not use for the initial pre-submission review — that is /paper-referee; this applies an EXISTING referee report''s findings.'

---

# paper-revision-response

Use this skill when the user asks to run the migrated source command `paper-revision-response`.

## Command Template


# /paper-revision-response <Axx|Mxx>

_Created: 07-07-2026 · Last updated: 05-09-2026_

The missing back half of [`/paper-referee`](https://github.com/gasyoun/claude-config/blob/main/commands/paper-referee.md): that skill produces
`<Axx>_review_<tier>.md` with numbered Major/Minor findings — this skill is what turns
that memo into applied fixes plus the point-by-point response document a venue or
co-author actually wants to see. ~18 papers have sat in "revision executed, pending
author pass" state; ≥15 sessions hand-rolled this shape (`A16_revision_majors6_8`,
`a44_fable_fixpass`, `a33_fable_polish`, …) before this skill existed.

Argument: the stable paper ID (`Axx`/`Mxx`) from `Uprava/ARTICLES.md`, or enough title
to resolve one.

## Phase 1 — Locate the review

Look the paper up in [`Uprava/ARTICLES.md`](https://github.com/gasyoun/Uprava/blob/main/ARTICLES.md):
repo, manuscript path, current readiness, blocker. Find the referee memo next to the
manuscript — `<Axx>_review_<tier>.md` (the `/paper-referee` naming convention). If no
memo exists, stop and say so: this skill applies an existing review, it does not run one
(that's `/paper-referee`).

Read the FULL memo and the FULL manuscript. Build a numbered worklist from every Major
and Minor finding in the memo, in the memo's own numbering — the response document must
map 1:1 onto it.

## Phase 2 — Apply fixes

For each finding, in memo order:

- If the fix is text-level and unambiguous, apply it directly to the manuscript.
- If a finding needs a new experiment, new data, or a scope/venue call only a human can
  make, do NOT invent a fix — leave it open and flag it as `@DECIDE` in Phase 4.
- Track, per finding: applied / not-applied (with reason), and the manuscript section
  touched.

Commit incrementally is fine, but the final state should be one branch ready for one PR
(per the large-work discipline — never push straight to a repo's default branch from an
unattended run unless it's the private Uprava hub).

## Phase 3 — Write the response document

Write `RESPONSE_TO_REVIEWERS_<Axx>.md` next to the manuscript, dated header + byline per
the doc contract, full blob URLs on every reference. One entry per finding, in the
memo's numbering:

```markdown
### Finding N (Major|Minor) — <one-line title>

> <verbatim quote of the finding from the review memo>

**Response:** <what was changed and why, or why it was left open>
**Change:** <full blob URL to the diff/commit, or "not applied — see @DECIDE below">
```

Close with a one-line tally: `N Majors applied / M open, P Minors applied / Q open`.

## Phase 4 — Deliver + bookkeeping

- Open the branch as a PR (title: `revision response: <Axx> — N Majors, M Minors
  applied`), auto-merge per the large-work rule.
- Any finding left open because it needs an MG ruling goes to
  [`Uprava/GTD_NEXT_ACTIONS.md`](https://github.com/gasyoun/Uprava/blob/main/GTD_NEXT_ACTIONS.md)
  as `@DECIDE`, referencing the response doc's entry.
- Bump readiness in `Uprava/ARTICLES.md` per the [`/articles-update`](https://github.com/gasyoun/claude-config/blob/main/commands/articles-update.md)
  discipline: all Majors applied and no open `@DECIDE` → 4/5 (pre-submission); any Major
  still open → stays at 3/5 and the blocker column names the open finding.
- Regenerate the dashboard (`python Uprava/tools/build_dashboard_data.py`), commit.
- Report: PR link, response doc link, applied/open tally, new readiness, any new
  `@DECIDE` rows.

_Dr. Mārcis Gasūns_
