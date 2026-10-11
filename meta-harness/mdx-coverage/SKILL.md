---
name: "mdx-coverage"
description: 'Audit a tree of .doc/.docx/.pdf sources for .mdx/.md siblings, batch-OCR any image-only PDF lacking a text layer, route gaps to the closing skill. Optional target dir, default SanskritGrammar. Triggers: which sources need OCR.'
---

# mdx-coverage

Use this skill when the user asks to run the migrated source command `mdx-coverage`.

## Command Template


# /mdx-coverage

_Created: 06-07-2026 · Last updated: 29-08-2026_

## Default path (seconds — run this first)

From the Uprava clone root:

```sh
python tools/mdx_coverage.py [TARGET_DIR]
```

LLM / judgment residue only after the script report exists (or the script exits non-zero with a real defect).

Audit a tree of source documents (`.doc` / `.docx` / `.pdf`) for whether each already has an
`.mdx` (or `.md`) sibling, and **batch-OCR any image-only PDF that has no text layer** so it can
be converted. Answers "which sources are still un-converted?" and "which scanned PDFs need OCR
before conversion?" in one pass, then points each gap at the skill that closes it.

The mechanics are a committed, deterministic script —
[`Uprava/tools/mdx_coverage.py`](https://github.com/gasyoun/Uprava/blob/main/tools/mdx_coverage.py)
(extracted from this skill's former inline program by
[H835](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H835-Opus_claude-config_skill-model-ver-fix-llm-to-py_13.07.26.md) —
Opus 4.8, the LLM→Python conversions handoff). This skill = the preflight, the
invocation, and how to route each gap.

**Argument:** an optional target directory (default: the SanskritGrammar checkout under your GitHub clones)
plus optional flags:

- `/mdx-coverage` — audit + OCR any text-less PDFs (when `ocrmypdf` is installed).
- `/mdx-coverage <path-to-a-clone>` — different tree.
- `/mdx-coverage --audit-only` — report coverage + which PDFs need OCR, but don't run OCR.

Read-first, then act: the audit is read-only; OCR only ever **writes a new `<name>.ocr.pdf`** beside
the source — it never edits or overwrites a source `.doc`/`.docx`/`.pdf`.

---

## What "has an mdx version" means

A source `X.doc` / `X.docx` / `X.pdf` is **covered** if `X.mdx` (or `X.md`) exists in the same
folder. `.doc` and `.docx` of the same stem are treated as one work — coverage is satisfied if
either has an `.mdx` (they usually share the basename, e.g. `Buhler_Unicode.doc` + `.docx` +
`.mdx`). A `.doc` with an authoritative sibling `.docx` counts as covered when the `.docx` has an
`.mdx`.

## What "no OCR" means for a PDF

An **image-only / scanned** PDF has no extractable text layer — selecting text yields nothing, and
a converter (Pandoc, `pdftotext`) gets an empty result. Detect it with `pdftotext` (poppler): if
the text extracted from the first pages is essentially empty, the PDF needs OCR before it can be
converted to `.mdx`. A PDF that already has a text layer ("has-text") is conversion-ready and is
**not** re-OCR'd.

## Phase 1 — Preflight

```bash
pdftotext -v 2>&1 | head -1              # REQUIRED for PDF text-layer detection (poppler)
command -v ocrmypdf && ocrmypdf --version | head -1 || python -m ocrmypdf --version 2>/dev/null   # needed only to run OCR; a pip install may be off PATH (the auditor falls back to `python -m ocrmypdf`)
command -v tesseract && tesseract --version | head -1
```

- **`pdftotext` missing** → install poppler (`choco install poppler -y`); PDF triage can't run without it.
- **`ocrmypdf` / `tesseract` missing** → OCR can't run. The audit still completes and lists what
  *would* be OCR'd. To enable OCR:
  ```
  choco install tesseract -y            # OCR engine
  pip install ocrmypdf                  # PDF OCR wrapper (needs Ghostscript: choco install ghostscript -y)
  ```
  Install the **Sanskrit + Russian + German** tesseract language packs (this corpus is Devanāgarī /
  IAST / Cyrillic / German): `san`, `rus`, `deu` (English `eng` ships by default). On Windows the
  packs are `tessdata` files under the tesseract install; verify with `tesseract --list-langs`.

## Phase 2 — Run the auditor + OCR batch

Run the committed script — do NOT re-emit the program inline:

```bash
python tools/mdx_coverage.py [TARGET_DIR] [--audit-only]
```

It walks the tree, classifies every source (`covered` vs missing `.mdx`), detects text-less PDFs
via `pdftotext` over the first 5 pages (`>= 40` non-whitespace chars = has a text layer), and —
unless `--audit-only` — OCRs each image-only PDF to `<name>.ocr.pdf` (langs `san+rus+deu+eng`,
resolving `ocrmypdf` on PATH or via `python -m ocrmypdf`). Re-runs skip a PDF that already has a
`.ocr.pdf`. Defaults to the SanskritGrammar checkout when no `TARGET_DIR` is given.

## Phase 3 — Report + route each gap

One short summary:

- **Coverage** — total sources, covered, and the list of **missing `.mdx`**, each routed to the
  skill that closes it: `.doc`/`.docx` → [`/docx-to-md`](https://github.com/gasyoun/claude-config/blob/main/commands/docx-to-md.md); a text-layer PDF →
  the pdf→mdx path (Pandoc/`pdftotext` + the same MDX-safety post-process); an image-only PDF →
  **OCR first**, then convert.
- **OCR** — which PDFs were OCR'd (`<name>.ocr.pdf` written), which failed, and — if `ocrmypdf`
  wasn't installed — the exact install commands + the list that still needs OCR.
- Flag any PDF whose text-layer detection returned `unknown` (encrypted / damaged / `pdftotext`
  missing) for a manual look.

## Notes

- **Never edits or deletes a source.** OCR output is a new `<name>.ocr.pdf`; conversion is a
  separate `/docx-to-md`-style step. Re-runs skip a PDF that already has a `.ocr.pdf`.
- **Text-layer heuristic:** `pdftotext` over the first 5 pages, `>= 40` non-whitespace chars = has
  text. A born-digital PDF with a real text layer passes; a pure scan yields ~0 and is flagged
  image-only. A few text-bearing cover pages in front of scanned body pages can produce a false
  "has-text" — spot-check a PDF that you expect to be a scan but reads as `has-text`.
- **The Cyrillic trap (never trust poppler on Russian PDFs).** `pdftotext` blanks Cyrillic text
  while returning plausibly-shaped output, so a Russian-text PDF can read as `image-only` and get
  OCR'd for nothing. Before OCR on any PDF you know carries Russian, verify with a second signal
  (open two pages visually, or check file provenance) — a false image-only verdict wastes an OCR
  pass; a real scan OCR'd twice is only redundant.
- **OCR language packs matter.** Devanāgarī needs tesseract's `san` pack; Russian needs `rus`;
  German (Bühler) needs `deu`. Missing a pack silently degrades that script's OCR to garbage —
  verify `tesseract --list-langs` before a large batch.
- **Related:** [`/docx-to-md`](https://github.com/gasyoun/claude-config/blob/main/commands/docx-to-md.md) converts the covered sources; [`/sanscritica-ocr`](https://github.com/gasyoun/claude-config/blob/main/commands/sanscritica-ocr.md)
  is the *vision*-OCR path for dictionary front matter (a different, higher-fidelity OCR for
  specific pages), not a batch text-layer OCR like this one.

_Dr. Mārcis Gasūns_
