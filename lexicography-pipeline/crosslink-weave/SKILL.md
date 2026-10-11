---
name: "crosslink-weave"
description: 'Turn a hub doc''s implicit references into a woven cross-link network — per-section anchors, internal [[...]]/GitHub evidence links, committed reproducible link-topology audit script. MEGABOOK weave series. Triggers: "crosslink weave", "link audit for X".'

---

# crosslink-weave

Use this skill when the user asks to run the migrated source command `crosslink-weave`.

## Command Template


# /crosslink-weave <hub-doc.md>

_Created: 11-07-2026 · Last updated: 25-08-2026_

Six handoffs in the MEGABOOK weave series (H617 source-ontology, H618/H619
attention/reuse maps, H622 research-product map, H624 wave-a integration, H626/H628
crosslink-weave proper) all ran the same uncovered procedure by hand: stabilize
anchors, count links per section, check for dead anchors — no shared script, no
skill. `/md-hygiene` doesn't cover this (it checks URL *form* — relative vs full
blob — and dated headers, never link *topology*). This skill is that procedure,
encoded once, with a reusable audit script instead of a fresh one-off count per doc.

Argument: the hub doc to weave (e.g. `MEGABOOK.md`, `ROADMAP_INDEX.md`), or infer it
from the conversation.

**Tier:** mechanical structure + judgment on which links are meaningful — Sonnet 5
(`claude-sonnet-5`); escalate to Fable 5 (`claude-fable-5`)/Opus 4.8 (`claude-opus-4-8`)
only when the doc's own content (which
claim needs which evidence) is genuinely open-ended, not just "count and link".

## Phase 1 — Pick the section depth, audit current state

1. Identify the heading depth that defines an "auditable section" — usually the
   thesis/topic level, one level under the doc's top grouping headers (MEGABOOK:
   `##` groups five areas, `###` are the 21 individual theses — the theses are the
   audit unit).
2. Run the audit script first, before touching content, to see the real gap:
   ```sh
   python Uprava/tools/crosslink_weave_check.py <hub-doc.md> --level 3
   ```
   (If the script isn't present yet in this checkout, it's committed at
   [`Uprava/tools/crosslink_weave_check.py`](https://github.com/gasyoun/Uprava/blob/main/tools/crosslink_weave_check.py)
   — pull it, don't re-derive a parser; see H747.)

## Phase 2 — Stable anchors

Every section at the chosen depth needs a **stable anchor** — GitHub's own
heading-slug algorithm (lowercase, Unicode letters/digits/spaces/hyphens kept,
everything else dropped, spaces→hyphens; `--list-anchors` on the script prints the
computed set for a doc). **Never rename an existing heading text if anything already
links to its anchor** — check `grep -rn "#<old-anchor>"` across the org before any
heading rewrite; if a rename is unavoidable, sweep every referrer in the same pass.

## Phase 3 — Weave: internal + evidence links per section

For each section under threshold (default: ≥3 internal, ≥2 evidence — both
arguments, not fixed):

- **Internal links** — `[[wikilink]]`-style or `text` to sibling
  sections in the same doc that the current thesis genuinely depends on or is
  depended on by, plus `text` cross-doc links to the doc's
  established companions (`PROJECT_INTERLINKS.md`, `REUSE_INDEX.md`, etc. for
  MEGABOOK). Meaningful "see also", not padding — a link that doesn't help a reader
  navigate the actual dependency is worse than no link.
- **Evidence links** — full GitHub blob/PR/commit/tree URLs (never relative, per
  the org clickable-links convention) that substantiate the claim the section
  makes: the PR that shipped it, the data file that backs a stated number, the
  script that computes a cited figure. A thesis with zero evidence links is an
  unfalsifiable claim — that's the defect this phase exists to close.
- **Volatile status stays out.** Live done/now/future state belongs in
  GTD/ARTICLES/REPO_STATUS, never inlined into the hub doc's stable prose — a
  crosslink-weave pass adds structure and evidence, it does not turn a stable doc
  into a second status tracker.

## Phase 4 — Re-run the audit, prove convergence

```sh
python Uprava/tools/crosslink_weave_check.py <hub-doc.md> --level 3 \
    --min-internal 3 --min-evidence 2
```

Exit 0 across every section = done. A second immediate re-run must report the
identical table (the audit is read-only, so this is automatic — no generator step
to prove idempotent here, unlike `/computed-table`).

## Acceptance

- `crosslink_weave_check.py --check` exits 0 for the target doc at the agreed
  thresholds, zero dead anchors.
- Every section passing has genuine (not padded) internal + evidence links —
  spot-check a few against the claims they're attached to.
- No existing anchor broken (verified via the pre-rename grep in Phase 2 if any
  heading text changed).
- Commit + push with model-tier provenance; PR for a shared/public repo, direct
  push for a private single-owner repo (Uprava takes direct `main` pushes).

## Guardrails

- **Read the doc's actual current sources before inventing links** — reuse
  `PROJECT_INTERLINKS.md`, `REUSE_INDEX.md`, `SHARED_CODE.md`, and the doc's own
  existing companion docs; don't derive a new crosswalk to link against.
- **Never fabricate an evidence link.** If a claimed thesis has no real committed
  artifact behind it yet, that's a finding to report (the claim is unsubstantiated),
  not a link to paper over with the nearest tangentially-related PR.
- **Shared-tree repos under concurrent minting** (Uprava, SanskritLexicography) —
  isolate in a worktree off `origin/main` with push-retry; never `git add -A` on
  the shared checkout.
- **Anchors are load-bearing** — a heading rename without a referrer sweep silently
  breaks every `#anchor` link pointing at it; the audit script's `--list-anchors`
  mode plus a repo-wide grep are the two required checks before any rename lands.

## Related

[`tools/crosslink_weave_check.py`](https://github.com/gasyoun/Uprava/blob/main/tools/crosslink_weave_check.py)
(the audit script this skill wraps), `/md-hygiene` (URL form + headers, the
complementary check this skill does NOT duplicate), `/computed-table` (same
"generated vs hand-authored" split, applied to tables instead of link topology),
`/metadoc` (companion-doc discipline for the same class of long-lived hub docs).

## When this isn't the right tool

- A doc with no stable thesis/section structure yet (still a flat list or a single
  block) — that's a restructuring task first, weaving comes after.
- Fixing URL *form* (relative→full blob, missing dated header) — that's
  `/md-hygiene`, not this skill.
- A one-off "add a link here" edit with no systematic per-section gap — just add
  the link, don't spin up the audit for a single fix.

_Dr. Mārcis Gasūns_
