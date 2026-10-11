---
name: "etext-acquire"
description: 'Acquire printed-edition e-text/OCR packs — library e-texts first, text-layer before re-OCR, quarantine/align/ingest, CER note, registry row. Rights uncertainty is not a stop (MG 30-07-2026). Triggers: "etext for Gorresio/Bombay/MBH", "OCR this edition".'

---

# etext-acquire

Use this skill when the user asks to run the migrated source command `etext-acquire`.

## Command Template


# /etext-acquire <work-or-edition>

_Created: 28-07-2026 · Last updated: 30-07-2026_

Skill-mine **C27** (H1661–H1778, MG vote 28-07-2026: **NEW separate skill**, not a mode of `/license-gated-ingest`). Encodes the edition e-text / OCR acquisition ladder that reappeared six times (Gorresio, Rāmāyaṇa Bombay, MBH Calcutta probe, PWG scan-index reuse, kosa e-text pilot, BSB hOCR align).

**Standing rights policy (github-wide, MG 30-07-2026):** rights uncertainty is **not** a stop. Record known rights facts once; do not park the acquire on `@DECIDE` for greyness. Canonical: [STANDING_POLICY_RIGHTS_UNCERTAINTY_IS_NOT_A_STOP_2026.md](https://github.com/gasyoun/Uprava/blob/main/docs/STANDING_POLICY_RIGHTS_UNCERTAINTY_IS_NOT_A_STOP_2026.md) · FINDINGS §255.

**Not this skill:**

| Need | Use instead |
|---|---|
| Generic inbound dataset + license facts | `/license-gated-ingest` (compose when recording license; **not** a park loop) |
| Cologne dictionary front matter OCR / scholarly scans | `/sanscritica-ocr` |
| Apparatus generation once text exists | `/critical-apparatus` |
| Publish the derived pack | `/data-release` + `/publish-safety-check` |

Argument: work/edition slug (e.g. `gorresio-vols-2-4`, `ramayana-bombay-book7`, `amara_dlc`, `mbh-calcutta`).

**Tier:** Opus 4.8 (`claude-opus-4-8`) for GO/NO-GO quality/prior-art judgment; Sonnet 5 (`claude-sonnet-5`) for mechanical hOCR align / index-anchored segmentation once the recipe is locked.

## Phase 0 — Prior art (blocking; re-OCR is the failure mode)

1. `/prior-art` against:
   - `sanskrit-lexicon-scans` (and Cologne scan READMEs)
   - Gretil / SARIT / other public e-texts already registered in PROJECT_INTERLINKS / kosha manifest
   - sibling FINDINGS (e.g. SL FINDINGS §59: re-deriving an existing Amarakoṣa e-text is a known defect)
2. If a **usable library e-text already exists** → written **NO-GO rebuild** + pointer to the existing asset is a successful outcome, not a failure.
3. `/source-health-probe` + `SERVER_OUTAGES.md` before hammering any origin.
4. Rights: record known facts once (or invoke `/license-gated-ingest` for the license note). **Grey redistribution does not block** continuing the acquire or committing derived measurements; raw bulk may stay gitignored for engineering reasons. Only a **confirmed prohibition** (or privacy / restricted designation) stops raw commit/publish.

## Phase 1 — Text-layer first, then OCR

Cheapest path wins:

1. **Embedded text layer** in library PDFs / Google Books / HathiTrust / e-rara (Gorresio vols with Google layer — H1656/H1689).
2. Else **modern OCR** on page images: tesseract 5 `san` or a Devanagari model; consumer may be n-gram fuzzy match (~85% char accuracy can suffice) — state the bar in the report.
3. Prefer **index-anchored segmentation** when a per-page scan index exists (PWG scan-index tracker / H1706–H1720): the index says what should be on each page; mis-segmentation becomes detectable.

Do **not** re-derive the OCR recipe from scratch — reuse FINDINGS + existing builders (`build-gorresio`, kosa hOCR align, etc.).

## Phase 2 — Ingest + measure

- Emit the repo's native layer format (jsonl / page-segmented text / hOCR → align).
- **CER or coverage** on a hand-checked sample (pilot: ≥30 pages when that is the stop condition; otherwise ≥10 spot-checks against scans).
- Update selftests / fixtures so a previously typed miss (e.g. `gorresio-etext-gap`) becomes a mapped gold when claimed done.
- Prefer gitignored raw bulk when large; commit derived layers freely. Confirmed prohibition blocks raw publish — uncertainty does not.

## Phase 3 — GO/NO-GO + registry

Write a short pilot note:

- source URL(s) + date + checksum
- method (text-layer vs OCR model)
- sample n + CER/coverage
- rights facts recorded (honest `unknown`/`mixed` is fine)
- **GO** extend queue / **NO-GO** (why — quality, prior e-text exists, confirmed prohibition — **not** mere greyness)
- registry: PROJECT_INTERLINKS / kosha manifest / scan-index tracker row as appropriate

## Hard rules

- Re-OCR when a library e-text already exists is a **census defect** — prior-art first.
- One edition (or one pilot work) per session; do not expand to the whole queue after a NO-GO.
- Rights grey → record once via `/license-gated-ingest` facts; **do not** open human `@DECIDE` solely for uncertainty.
- `/publish-safety-check` before anything public (same risk-tolerant rights rule).

_Dr. Mārcis Gasūns_
