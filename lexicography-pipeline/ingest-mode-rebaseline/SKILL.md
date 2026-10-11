---
name: "ingest-mode-rebaseline"
description: 'Turn recurring ingest parse defects into opt-in parser MODES + re-baseline affected works: before/after tables, round-trip >=99%, sibling census; old default kept for siblings. Mode Opus 5 (claude-opus-5); re-baselines Sonnet 5 (claude-sonnet-5).'

---

# ingest-mode-rebaseline

Use this skill when the user asks to run the migrated source command `ingest-mode-rebaseline`.

## Command Template


# /ingest-mode-rebaseline [defect | work | --census]

_Created: 12-08-2026 · Last updated: 26-08-2026_

Skill-mine **D58**, **force NEW** on MG's vote (12-08-2026): the census had filed the glued-digit rebaseline as "one corpus defect class in one repo, reopen if it appears in a second" and the human **rejected that affirmation**, which is the instruction to build it now rather than wait — the same call that produced [`/start-chteniya`](https://github.com/gasyoun/claude-config/blob/main/commands/start-chteniya.md) from D43 last census. Census: [SKILL_CANDIDATES_CENSUS_H2323_H2591_12_08_2026.md](https://github.com/gasyoun/Uprava/blob/main/archive/SKILL_CANDIDATES_CENSUS_H2323_H2591_12_08_2026.md); built as [H2604 (Opus 4.8) — claude-config skill-build D58 OCR-defect rebaseline](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H2604-Opus_claude-config_skill-build-d58-ocr-defect-rebaseline_12.08.26.md).

Argument: the defect signature, one work to re-baseline, or `--census` to sweep the remainder.

## The defect class

An ingest parser handles **one** convention and silently mangles a text that uses another. The worked example: page-local **glued-digit superscript footnotes**, where the generalized parser's `_NOTES_HEAD_RE` looked only *after the last chapter*, so per-page footnote blocks read as body text and footnote citation numbers (`1.1(2)`) were misread as **verse boundaries**.

The symptoms are diagnostic, and they look like data problems rather than parser problems:

| Symptom | What it actually means |
|---|---|
| `verse_count` oscillates between re-runs (1↔2) | citation numbers are being read as boundaries |
| a bogus jump (`2 -> 1772`) | a footnote block parsed as verse ids |
| `comment_count: 0` on a book that visibly has footnotes | the notes were never extracted at all — **the loudest tell for the census** |
| wild `verse_gaps` / `id_collisions` | both of the above |

## Phase 0 — Design first, ingest second

**This is a design task before it is an ingest.** The fix is an additional **front-end mode** (an explicit flag, plus optional per-work auto-detect) — *not* a one-line regex tweak on the shared path. A tweak that makes this book parse will change how every already-baselined book parses, which is the failure mode Phase 3 exists to prevent.

If the design needs a human ruling, write the options memo and stop there. That is a complete session.

Check prior art first (`/prior-art` + the repo's own pipeline doc): the glued-digit heuristic already existed in an older path and was reintroduced as a mode, not invented.

## Phase 1 — Diagnose on ONE work, hermetically

Pick the work that surfaced the defect and pin the diagnosis in the pipeline doc (the org's case: [SamudraManthanam `web/corpus_builder/PDF_INGESTION_PIPELINE.md`](https://github.com/gasyoun/SamudraManthanam/blob/main/web/corpus_builder/PDF_INGESTION_PIPELINE.md) § Wave A) so the next session does not re-derive it.

Then write **hermetic unit tests covering the false-split class** measured on that work — small fixtures, no archive dependency. The tests are what let the mode ship without fear.

## Phase 2 — Ship the mode opt-in, auto stays the old default

```
--footnote-mode glued-digit     # forced, per work
auto                            # unchanged: still bracket
```

**`Fail =` a regression on other works without opt-in.** The default must keep producing byte-identical output for every already-baselined text; that is the invariant that makes the rest of the fan-out safe.

## Phase 3 — Re-baseline the siblings, one handoff per work

Once the mode lands, every affected work gets its **own** handoff (`/handoff-mint --batch` — N are known at once). Each one delivers:

- raw/aligned JSONL + report + HTML + meta + **manifest pin**
- a **before/after table**: `verse_count` / `comment_count` before → after. This is the deliverable, not a footnote. Real examples: Nirvāṇa `465/0 → 527/212`, Yoni `221/0 → 221/192` (note the second: verse count *correctly unchanged*, comments recovered — an unchanged count is a valid result and must not be "improved").
- **stability**: re-parse the same extract, `verse_count` must be **equal**. Oscillation was the original symptom, so proving it stopped is proving the fix.
- **round-trip ≥99%** HTML→JSONL.
- reconciliation with any prior count-drop justification doc — an older explanation of a count usually stays valid for the intermediate path and should be linked, not deleted.

`Fail =` silent empty chapters · a sibling regressing without opt-in · a count that moved with no before/after row explaining it.

## Phase 4 — Census the remainder

Sweep every remaining work for the same signature — `comment_count: 0` against a text known to carry notes is the cheap tell — and report which works are affected, which are clean, and which are indeterminate. **Name what you did not check.** A census that quietly covers only the easy half reads as "all clear".

New works found affected become new re-baseline handoffs, not scope added to the current one.

## Phase 5 — Propagate

- Pipeline doc: the mode, its flag, its auto-detect rule, and the defect signature.
- Dataset/manifest changes → [kosha `datasets.json`](https://github.com/gasyoun/kosha/blob/main/data/manifest/datasets.json) row in the same pass, per the org rule.
- Measured caveats → [`/findings-append`](https://github.com/gasyoun/claude-config/blob/main/commands/findings-append.md) (Sanskrit-data half).
- Counts that changed → the results table lands in the repo doc **and** the tracking issue.
- `CHANGELOG.md` + [`/cut-release`](https://github.com/gasyoun/claude-config/blob/main/commands/cut-release.md).

## Hard rules

- **Design the mode before touching the shared path.**
- **Opt-in only**; `auto` keeps the old behaviour.
- **Hermetic tests for the false-split class** before the fan-out.
- **Before/after table per work**, including the works whose counts correctly did not move.
- **Prove stability across re-run** — the oscillation is the defect.
- **Census names its own gaps.**

## Not this skill

| Situation | Use instead |
|---|---|
| A canonical store is corrupted and needs evidence-preserving repair | [`/canonical-data-repair`](https://github.com/gasyoun/claude-config/blob/main/commands/canonical-data-repair.md) |
| Map causes/consumers of a pipeline before changing it | [`/pipeline-audit`](https://github.com/gasyoun/claude-config/blob/main/commands/pipeline-audit.md) |
| Judge whether the repaired pipeline is ready | [`/acceptance-gate`](https://github.com/gasyoun/claude-config/blob/main/commands/acceptance-gate.md) |
| Harden a parser's round-trip invariants generally | [`/parser-roundtrip-harden`](https://github.com/gasyoun/claude-config/blob/main/commands/parser-roundtrip-harden.md) |
| Acquire the source text in the first place | [`/etext-acquire`](https://github.com/gasyoun/claude-config/blob/main/commands/etext-acquire.md) |

_Dr. Mārcis Gasūns_
