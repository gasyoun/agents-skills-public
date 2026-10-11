**`feat/ocr-v2-pipeline` is the PD (Deccan College) feature branch** (Grok-side Tesseract v2: crop-then-OCR, layout modes `multi_pair` / `multi_seq` / `prose_2col`). On CDSL it is **not** a second author — it is Phase 3.5 comparison/QA. See [`PD/README.md`](https://github.com/sanskrit-lexicon/PD/blob/main/README.md), [`PD/compare_pwg_a_vs_b.py`](https://github.com/sanskrit-lexicon/PD/blob/main/compare_pwg_a_vs_b.py), and the PWG bake-off [`PD/COMPARISON_PWG_OCR_A_VS_B.md`](https://github.com/sanskrit-lexicon/PD/blob/main/COMPARISON_PWG_OCR_A_VS_B.md) (A wins for edition quality; B useful for coverage/layout flags).
### What to run

1. For each page (or a stratified sample: 1 title + 1–2 prose + 1–2 abbrev/addenda when the dict is large), run Tesseract crop-then-OCR on the same PNG as A:
   - **Title / centered single-column** → `single` (full page, margins trimmed).
   - **Foreword / prose 2-col** → `prose_2col` (LEFT then RIGHT × horizontal bands).
   - **Abbreviation / addenda grids** → same L→R band crop (or PD `multi_pair` / `multi_seq` only when the source is a PDF table, not csldoc PNG).
2. Language pack: prefer `deu+eng+san` for German sources, `eng+san` for English, `fra+eng+san` for French if installed, else `eng+san`.
3. Reference implementation (PWG sample bake-off — adapt paths/manifest for the current `CODE`):
   ```text
   # from PD repo; generalize SAMPLES to the dict's prefNN ↔ scan map
   python ~/Documents/GitHub\PD\compare_pwg_a_vs_b.py
   ```
   Or inline the same logic: crop → `pytesseract.image_to_string` per band → write `prefaces/_audit_b/<code>prefNN.b.md` (gitignored or deleted in Phase 7 unless the user wants the audit kept).

### Metrics (per page, A as reference)

Compute after normalizing: lowercase; drop A’s `**bold**` / `` `code` ``; drop `[page N]` markers.

| Metric | Formula | How to read |
|---|---|---|
| **Recall@A** | \|tokens(A) ∩ tokens(B)\| / \|tokens(A)\| | Low → B missed a lot of A’s vocabulary **or** A has long invented spans B never saw — investigate both ways |
| **Jaccard** | \|A ∩ B\| / \|A ∪ B\| | Overall vocabulary overlap |
| **Length ratio** | `chars(B) / chars(A)` (bodies only) | ≪ 0.7 → possible A over-write / B crop miss; ≫ 1.3 → B noise or A skip |

PWG reference means (3 layout types, 23-07-2026): Jaccard ~0.51, Recall@A ~0.72; **prose best** (~0.73 / 0.92), **title & abbrev worst** (~0.41). Do not expect English-PDF-table scores on German/Sanskrit-heavy CDSL pages.

### Emit `prefaces/AUDIT_A_VS_B.md`

One file per dictionary (append-friendly if re-run). Suggested shape:

```markdown
# Audit A (vision) vs B (Tesseract) — <CODE>

_Date: DD-MM-YYYY · Engine B: tesseract-… lang=… · pipeline: crop-then-OCR_

| NN | Section | Layout | Recall@A | Jaccard | len(B)/len(A) | Flag |
|---|---|---|---:|---:|---:|---|
| 02 | Foreword, 1 | prose_2col | 0.91 | 0.73 | 1.04 | ok |
| 07 | Abbreviations, 1 | prose_2col | 0.63 | 0.41 | 0.99 | review keys |

## Flags (actionable only)

### prefNN — <reason>
- Signal: e.g. "B has dense tokens in right-col lower band; A body ~40% shorter"
- Action: re-crop + re-Read that band (Phase 3); do **not** paste B.
- Resolution: fixed / false-positive (B garbage) / deferred
```

**Flag when any of:**

| Signal | Likely cause | Action |
|---|---|---|
| A long prose, B nearly empty in a band | A invention, or B crop/threshold fail | Re-vision that band; if A still matches scan, mark B false-positive |
| B dense, A short for same page | A skipped column/band | Re-OCR missing region with Phase 3 crops |
| Both disagree on a **number / date / volume** | Either engine | Re-vision the locus; trust the scan |
| Disagreement only on diacritic-heavy **abbrev keys** | B weakness (expected) | Do **not** flag as A error unless visual re-check fails |
| B-only digitizer footer/header | Stamp | Ignore |

### After the audit

1. For each open flag: re-run Phase 3 **only on the flagged band/page** (native-res crops).
2. Update `<code>prefNN.md` from vision only; re-run B on that page if you want a clean re-score.
3. Mark flags resolved in `AUDIT_A_VS_B.md`.
4. Proceed to Phase 4. Link the audit from `prefaces/README.md` (one line under Notes) when it found real fixes; optional otherwise.
5. Phase 7: delete `prefaces/_audit_b/` raw B dumps unless the user asked to keep them; **keep** `AUDIT_A_VS_B.md` if any flag was non-trivial (durable QA).

### What Phase 3.5 is *not*

- Not a license to merge B into A automatically.
- Not required to re-OCR every Done dict retrospectively (optional backfill).
- Not a substitute for Phase 3 on remaining queue (AP90, BUR, …) — still author with A first.
