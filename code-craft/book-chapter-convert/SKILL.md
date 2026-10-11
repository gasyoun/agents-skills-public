---
name: "book-chapter-convert"
description: 'Article-to-monograph-chapter conversion, M01 discipline: strip apparatus, provenance headnote, register unify, crosswalk weave, counts unchanged, one merged PR. Opus 4.8 (claude-opus-4-8); interleave Fable 5 (claude-fable-5). Do not use to draft original new chapter prose — that is /book-chapter-drain; this only converts an EXISTING article into monograph-chapter form.'

---

# book-chapter-convert

Use this skill when the user asks to run the migrated source command `book-chapter-convert`.

## Command Template


# /book-chapter-convert <book-dir> ch## <Axx[+Ayy]>

_Created: 17-07-2026 · Last updated: 05-09-2026_

Distills the 13-07-2026 M01 conversion wave (11 Opus 4.8 sessions: H846 H851 H853 H854 H857 H860 H861 H863 H865 H866 H867, SanskritLexicography) that took the Brill monograph *Digital Sanskrit Lexicography* from 2 to 12 of 14 chapters in book form — one chapter per session, same moves every time. Dedupe: [/docx-to-md](https://github.com/gasyoun/claude-config/blob/main/commands/docx-to-md.md) owns format-conversion mechanics (Word→MDX); [/book-press-prep](https://github.com/gasyoun/claude-config/blob/main/commands/book-press-prep.md) owns camera-ready print prep. This skill owns the step between: turning a standalone journal article's prose into a chapter that belongs to a book.

## The invariant that defines the skill

**Every count and table carried over unchanged.** Conversion touches apparatus, register, references, and grounding — never the empirical substance. Diff every number against the source before delivering. Draft placeholders (`[author to verify …]`) are resolved into *honest footnotes* that state what is asserted vs. measured — never into invented facts (the [/book-press-prep](https://github.com/gasyoun/claude-config/blob/main/commands/book-press-prep.md) guardrail, inherited whole: mechanics only; where substance is missing, report, don't fabricate).

## Phase 0 — Plan of record + sources

- Read the book's plan-of-record (M01: [BOOK_PLAN.md](https://github.com/gasyoun/SanskritLexicography/blob/master/Digital_Sanskrit_Lexicography-BOOK/BOOK_PLAN.md) + its `.meta.md`) — confirm the chapter←article mapping, any per-chapter mandate (merge, reframe), and the progress table you'll bump.
- **Verify the source path is live.** Sources are often cross-repo (csl-atlas `docs/articles/`) and the wave found two source-path staleness bugs (H867) — confirm you have the article's current committed version, not a stale copy.
- **Classify before converting:** a prose-thin data chapter (A40/A50-class) is NOT a conversion — it needs new prose atop committed data ("genuine drafting load"). Flag it back as its own session; don't fake a conversion.
- Usual contention checks: `git fetch origin`, `.ai_state.md` WIP, claimed registry rows; worktree if the repo is commit-guarded.

## Phase 1 — Conversion moves (the mechanics)

Strip the journal apparatus: abstract, keywords, venue/status box, ORCID byline, "byline to finalise", internal planning memos, "to do before submission" blocks — folding anything load-bearing (an abstract's framing sentence) into the chapter opening rather than deleting it. Unify register to the book's voice (M01: first person). Add the **provenance headnote**: the article stays independently citable at its venue — name the venue; for merges, flag the dual sourcing.

## Phase 2 — Remap references

Companion-paper references ("P-numbers", sibling-article citations) become sibling-chapter cross-references (the A-ID-not-P-number contract, BOOK_PLAN.meta.md). Papers absorbed into another chapter are noted as absorbed, not cited as if standalone. Every remap resolves to a chapter that actually exists in the plan.

## Phase 3 — Crosswalk weave

Read the book's literature crosswalk for this chapter's mandated grounding and weave the secondary sources into the prose — as genealogy, contrast cases, or the theory the result *tests*, not as a bolted-on reviews paragraph. **Mandatory reframes are non-optional** (H861/H866: a crosswalk row saying "reframed as an empirical test of X" must be executed, not cited). Honesty clauses survive the weave (H846: "neither comparandum has a sexual-taboo screen" — keep the limits of the comparison explicit).

## Phase 4 — QA + merge chapters

- Per-chapter QA: every count/table diffed unchanged against source(s); placeholders resolved honestly; cross-references resolve; register consistent with the already-converted chapters.
- `Axx+Ayy` merge chapters are **true interleaves, not concatenations** (H866: Part I/Part II by *axis of analysis*, both halves reframed against the shared theory) — this is the judgment-heavy variant; both articles stay independently citable in the headnote.

## Phase 5 — Records sweep + deliver

Same pass, never deferred: plan-of-record progress table (n/N in book form), `CHANGELOG.md` `[Unreleased]`, `.ai_state.md` next-steps re-queue, the plan's `.meta.md` backlog tick if one exists. One chapter = one PR, auto-merged; note the goal-loop remainder (which chapters are left and which are the special cases) in the handoff/`.ai_state.md`. Model tier + exact version stated.

## Gotchas

- Book dirs can live behind gitignore allow-lists or on a non-main branch — verify before writing (the /book-press-prep Phase-0 lesson).
- Mid-wave rulings (an `@DECIDE` resolved in chat) get applied to the crosswalk/plan in the same PR — don't let a ruling live only in the conversation (H846).
- Evidence: H846 H851 H853 H854 H857 H860 H861 H863 H865 H866 H867 (11 sessions, Opus 4.8 `claude-opus-4-8[1m]`, SanskritLexicography).

_Dr. Mārcis Gasūns_
