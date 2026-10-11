---
name: "book-chapter-drain"
description: 'Drain manuscripts chapter-by-chapter against the plan: harvest sources, recompose original prose, MG visa cycle, typeset, one PR per chapter, tick registry; transforms go to /book-chapter-convert. Trigger: "write chapter N". Do not use for the article-to-chapter transform step alone — that is /book-chapter-convert (invoked from within this drain); use this to run the chapter-by-chapter authoring loop.'

---

# book-chapter-drain

Use this skill when the user asks to run the migrated source command `book-chapter-drain`.

## Command Template


# /book-chapter-drain <repo>/<book-dir> [chapter]

_Created: 17-07-2026 · Last updated: 05-09-2026_

Distilled from two independent chapter-authoring programmes that had each been re-deriving the
same loop by hand: the **M03 monograph** (SanskritGrammar `GasunsDhatu_2014/revision-2026/`, 14
sessions — [H991](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H991-Opus_SanskritGrammar_m03-ch1-ch2-recompose-execute_15.07.26.md) (Opus 4.8, M03 Гл.1+Гл.2 recompose)/[H992](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H992-Opus_SanskritGrammar_m03-ch1-prose-rework_15.07.26.md) (Opus 4.8, M03 Гл.1 prose rework) recomposing Гл.1/Гл.2 by the three-question spine, [H1069](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1069-Fable_SanskritGrammar_m03-part1-core-rewrite-wave1_16.07.26.md) (Fable 5, M03 part-1 core rewrite wave 1)'s checklist, the
"было 2014 / стало 2026" provenance-footnote convention) and the **Brill journal→book conversion**
(SanskritLexicography `Digital_Sanskrit_Lexicography-BOOK/`, 10 sessions, H846–H867). Both are
"read the plan, drain one chapter, visa, typeset, tick, PR" — this skill is that shell.

**Dedupe against `/book-chapter-convert` (already built from the same Brill H846–H867 evidence,
found on prior-art check after this skill's first draft):** that skill already owns the SPECIFIC
mechanical journal-article→chapter transform (strip journal apparatus, provenance headnote,
register unification, cross-reference remap, crosswalk-weave, carry every number/table unchanged).
Do not re-derive that transform here. When the chapter IS a journal-article conversion, Phase 2
below is `/book-chapter-convert` — invoke it for the transform itself, then continue with this
skill's Phase 3–5 (visa/typeset/tick) around its output. This skill's own Phase 2 (recompose/author
from scratch) is for the M03-style case: an ORIGINAL chapter with no single published article
behind it, which `/book-chapter-convert` was never built to handle.

Argument: the repo + book directory (its own `BOOK_PLAN.md` or `LITERATURE_CROSSWALK.md` is the
plan of record), and optionally which chapter — default to the plan's own next-undrained chapter.

## Phase 0 — Read the plan, not a summary of it

Open the book's own plan doc (`BOOK_PLAN.md`, `IMPROVEMENT_ROADMAP.md`, or the journal-conversion
crosswalk) and find the target chapter's **current state**: still original text, partially
recomposed, or already visaed. Never assume a chapter's status from a handoff title — the plan
doc's own per-chapter checklist row is authoritative (M03's own history shows a session's first
draft of a diff sometimes paired one snapshot's numbering with another's — read the plan fresh).

## Phase 1 — Harvest the chapter's sources

Gather everything the chapter's new content depends on: research handoffs closed since the last
pass, data files/scripts the chapter cites, the prior edition's own text for what's being
superseded (never silently dropped — see Phase 2), and for a journal→book conversion, the
published article(s) mapped to this chapter number in the crosswalk. State the source list before
writing — a chapter recomposed from an incomplete harvest is the failure mode this phase guards.

## Phase 2 — Recompose / author (or delegate to /book-chapter-convert)

**If this chapter is a journal-article conversion** (a published `Axx` paper mapped to this chapter
number in the book's own crosswalk), stop here and run `/book-chapter-convert <book-dir> ch## <Axx>`
instead of hand-authoring — it already owns this exact transform. Resume this skill at Phase 3 with
its output.

**Otherwise (M03-style original chapter, no single published article behind it):** this is the one
phase `/book-press-prep` explicitly forbids itself from doing — here it is the point. Rewrite the
chapter's prose against the harvested sources, following the book's own established voice/method
(M03's three-question spine: what did 2014 claim, what does 2026 data show, what changed) rather
than inventing a new structure per chapter. Every claim/number that changed from a prior edition
gets a **provenance footnote** ("было 2014 / стало 2026"-style) — the edition boundary must stay
visible, not silently overwritten, both for the author-visa reviewer and for a future reader
comparing editions. Do not fill a gap with an invented number; flag it for the harvest phase or the
author instead (same guardrail `/book-press-prep` states for its own scope).

## Phase 3 — Author visa

The chapter is a draft until the author (MG) visas it — never treat a recomposed chapter as final
on its own authority. Package it the same way other visa cycles in this org do: a `/review-sheet`
if the chapter has enough discrete decision points to warrant per-item voting, or a direct
draft-for-review handoff if it's a single up/down call on the whole chapter. Record the visa
verdict in the plan doc's own checklist row, not just in a handoff.

## Phase 4 — Typeset + land

Once visaed, run the repo's own conversion/typesetting step (hand off to `/book-press-prep` if the
repo has one — mechanics only, do not re-invoke authoring). Land as **one PR per chapter** (never
batch multiple chapters into one PR — each needs its own visa trail and review surface). Verify a
site-bearing repo's build stays green.

## Phase 5 — Tick the plan + sweep the registry

Update the book's own plan doc (checklist row → done, with the PR link) in the SAME commit as the
landed chapter — a plan doc that lags the actual chapters drained is exactly the staleness class
this skill exists to stop. If the chapter closes a paper/thesis deliverable tracked in
`Uprava/ARTICLES.md`, bump its readiness there too.

## Output

Report per chapter: source list (one line), what changed vs the prior edition, visa status, PR
link, and the plan doc's updated row. A chapter "done" with no PR link, or a plan row left stale,
is not delivered.

_Dr. Mārcis Gasūns_
