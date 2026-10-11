---
name: "sandhi-annotate"
description: 'Batch-annotate every sandhi junction in a Sanskrit text inline — induced rule + corpus frequency per boundary — emitting a theme-aware reading edition. Sonnet 5 (`claude-sonnet-5`) tier. Triggers: "sandhi-annotate", "sandhi reading edition". Do not use to validate correctness (use /sandhi-check) or to split unsegmented input (use /sandhi-split) — this only annotates an already-segmented text for reading.'

---

# sandhi-annotate

Use this skill when the user asks to run the migrated source command `sandhi-annotate`.

## Command Template


# /sandhi-annotate <text | file>

_Created: 14-07-2026 · Last updated: 05-09-2026_

The static, whole-text counterpart to `/sandhi-split` (one string) and the reader-hover
surface ([H917](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H917-Opus_SanskritGrammar_sandhi-reader-hover-collider_14.07.26.md), Opus 4.8 `claude-opus-4-8`). Built on [SANDHI_PROGRAMME.md](https://github.com/gasyoun/kosha/blob/main/SANDHI_PROGRAMME.md).

## Procedure

1. **Get the split per sentence.** If the text is in DCS, use the gold `Unsandhied=` (method A). Else segment with method C (`/sandhi-split`'s caller).
2. **Induce every junction** via `dcs_sandhi_induce`'s mode-1/2/2b — the same machinery that built the corpus tables.
3. **Look up** each rule's frequency + class + rank from `corpus_sandhi.tsv`, and its **curriculum lesson** from `sandhi_curriculum.tsv` ("you'll meet this in lesson 2").
4. **Emit** a self-contained, theme-aware HTML edition: the running text with each junction underlined/tagged; hover or footnote shows `word₁ + word₂ → surface (rule · class · #rank · lesson N)`. Mirror the existing `reading/sandhi/` page style.
5. Register the edition under `reading/sandhi/editions/<text>/` if it's a keeper.

**Guardrail:** flag junctions where the split was method-C (lower confidence) distinctly from gold-split ones, so the reader knows which annotations are certain.

_Dr. Mārcis Gasūns_
