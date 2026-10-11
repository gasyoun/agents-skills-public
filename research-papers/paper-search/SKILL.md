---
name: "paper-search"
description: 'Search academic papers over arXiv + Semantic Scholar — deduped results as ranked table, BibTeX, JSON, or literature-matrix TSV/MD. Best for computational-Sanskrit literature (arXiv/S2 miss JSTOR/Brill venues). Triggers: "paper search", "literature matrix". Do not use to assemble an existing paper''s submission package — that is /paper-submission-pack; this only searches arXiv/Semantic Scholar.'

---

# paper-search

Use this skill when the user asks to run the migrated source command `paper-search`.

## Command Template


# /paper-search &lt;query&gt; [--source arxiv|s2|both] [--limit N] [--sort relevance|newest] [--bibtex | --json]

_Created: 16-07-2026 · Last updated: 05-09-2026_

The zero-dependency scholarly-search capability, built to replace the marketplace
`paper-search-usage` skill — which packaged the **same two databases** behind an opaque
`mcp/paper-search` Docker image (skill-fit audit
[H999 (Opus 4.8) — Uprava skill-fit-audit of the claudeskills marketplace](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H999-Opus_Uprava_skill-fit-audit-claudeskills_16.07.26.md)
→ **HOLD**). Both public APIs are queried directly by
[`Uprava/tools/paper_search.py`](https://github.com/gasyoun/Uprava/blob/main/tools/paper_search.py) —
no Docker, no MCP server, no API key.

## Run

```bash
python Uprava/tools/paper_search.py "Sanskrit sandhi segmentation"
python Uprava/tools/paper_search.py "vedic accent" --source arxiv --sort newest --limit 15
python Uprava/tools/paper_search.py "Sanskrit morphology" --bibtex      # → paste into a .bib
python Uprava/tools/paper_search.py "Panini grammar" --json             # → machine-readable
```

- **Sources:** `arxiv` (Atom API), `s2` (Semantic Scholar Graph API), or `both` (default; merged + de-duped by normalized title).
- **`--sort newest`** fields every term to all-fields and ANDs them (a bare `all:<multi word>` fields only the first token and, under newest-sort, floods with irrelevant recent papers — the script guards this).
- **`--bibtex`** emits ready-to-paste entries — pairs with `/bib-hygiene` and `Uprava/ARTICLES.md`.
- **Literature matrix (say `--matrix` or ask for it; [H1584 (Opus 4.8) — Uprava skill-consolidation/memory/academic-writing wave](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1584-Opus_Uprava_skill-consolidation-memory-academic-wave_24.07.26.md))** — NOT a script flag (`paper_search.py` implements no `--matrix`; verified 26-08-2026): after the ranked table, the session itself also writes a screening matrix next to the paper (or under `Uprava/scratch/` if no paper path): columns `cite_key | year | venue | method_one_line | data/assets | relevance (H/M/L) | keep? (Y/N/?) | notes`. Fill from titles/abstracts only unless the user asks for deep reads (`deeppapernote` / full PDF). Cap at the same `--limit`. This is the house form of the marketplace `literature-scout` capability — **no third-party import**.

## Notes

- **Semantic Scholar rate-limits unauthenticated calls (HTTP 429).** The script retries with backoff and, if S2 still fails, **still returns arXiv** (graceful degradation). Set `SEMANTIC_SCHOLAR_API_KEY` (or `S2_API_KEY`) to raise the limit.
- **Coverage is STEM-skewed.** Excellent for computational Sanskrit (the A/M-series method papers — segmentation, morphology, meter, OCR, MT); it will NOT surface core philology/lexicography scholarship in JSTOR / Brill / indological journals. For those, use the library databases directly.
- **Provenance:** capability adopted from the audit's `paper-search-usage` HOLD, but re-implemented in-house (stdlib, no Docker) rather than importing the third-party Docker MCP. Org-authored — not in the 📥 Imported-skills section.

_Dr. Mārcis Gasūns_
