---
name: "entry-specimen"
description: 'Visual sibling of /entry-anatomy: Duden-style annotated entry specimen, SVG leader-line callouts, print PDF + interactive page; --image/--markup modes. Fable 5 (claude-fable-5), Opus 4.8 (claude-opus-4-8) fallback. Trigger: "entry specimen".'

---

# entry-specimen

Use this skill when the user asks to run the migrated source command `entry-specimen`.

## Command Template


# /entry-specimen

_Created: 13-07-2026 · Last updated: 29-08-2026_

The **visual** counterpart to [`/entry-anatomy`](https://github.com/gasyoun/claude-config/blob/main/commands/entry-anatomy.md). Where `/entry-anatomy`
tints structural parts inline (per-headword HTML+JSON off the CDSL markup — a *textual*
breakdown), `/entry-specimen` produces the thing Fable 5 (claude-fable-5) made from the Duden *Deutsches
Universalwörterbuch* spread ([`papers/duden_deutsches_universalworterbuch-page.pdf`](https://github.com/gasyoun/SanskritLexicography/blob/master/papers/duden_deutsches_universalworterbuch-page.pdf)):
**callout labels wired by leader lines to highlighted spans of an entry**, with a facsimile
inset and a print-ready single sheet — the "commented on the picture" asset, not the tinted
text one.

**This generalizes [H780](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H780-Fable_SanskritLexicography_entry-anatomy-specimen-pages_12.07.26.md)
— do NOT reinvent its engine.** [`SanskritLexicography/EntryAnatomy/build_entry_anatomy.py`](https://github.com/gasyoun/SanskritLexicography/blob/master/EntryAnatomy/build_entry_anatomy.py)
already owns the reusable core: the `.callout` div + `data-target` anchor model, the
`svg.leaders` layer that draws leader-line `<path>`s from each callout to its target on load,
the `@page` auto-sizing that makes a single-sheet PDF regardless of height, `callout_html()`,
`wrap_once()` (span highlighting), `data_uri()` (facsimile embedding), and `page()`. Consume
that engine; the skill only adds the two new capabilities below.

## Arguments

- **Mode (one required):**
  - `--image <path>` — annotate a **supplied picture**: a dictionary scan, a PDF page (rasterized), or a screenshot. Works for ANY dictionary (Duden included), no markup needed. Callout anchors are regions on the image.
  - `--markup <dict> <headword>` — re-typeset an entry from `csl-orig/v02/<dict>/<dict>.txt` print-faithfully (the H780 path, generalized from the fixed *heman*/*gup* exemplars to any `<k1>` headword), with an optional facsimile inset from the Cologne scan server.
- **Output (default: both):** `--pdf` (print-faithful single sheet) · `--web` (interactive theme-aware page). Omit → emit both from one source.
- `--callouts <spec>` — the label set (see Phase 2). `--out <dir>` · `--title <text>` · `--facsimile <img>` (markup mode inset).

## Phase 1 — Establish the target surface

- **`--image`:** load the picture; if a PDF page, rasterize to PNG at print DPI first. Embed as a `data:` URI (per `data_uri()`), so the output is self-contained. The image is the annotation surface; callout targets are `{x,y,w,h}` regions (fractions of image size) the leader lines point into.
- **`--markup`:** reuse the engine's `load_records` → `parse_header`/`render_body`/`mw_paragraph`/`pwg_paragraph` to re-typeset the requested headword's records; wrap each part-to-label in a `wrap_once()` span so it becomes a leader-line target. Pull a facsimile crop (the Cologne scan-server direct-PDF pattern noted in H780 — `PWGScan`/`MWScan` `servepdf.php`) when `--facsimile` isn't supplied.

## Phase 2 — Author the callouts (the judgment core)

Each callout = `{ side (left|right), target (span id or image region), label, note }`. This is
where the skill earns its Fable 5 (claude-fable-5) tier: choosing *which* structural elements to teach and naming
them precisely — homograph indices, script levels, accent marks, gloss-language, exact-locus
citations with run-on abbreviation, grammatical apparatus, nested prefixed verbs (the PWG/MW
inventories in the [`EntryAnatomy/README.md`](https://github.com/gasyoun/SanskritLexicography/blob/master/EntryAnatomy/README.md)
are the reference vocabulary). Provide callouts via `--callouts` (a small JSON/TSV: label →
target → side) or, absent a spec, propose a first pass from the entry's structure and mark it
"proposed — verify" (never present an invented anatomy as authoritative). Keep sigla verbatim.

## Phase 3 — Render both outputs from one source

- **`--pdf`:** the H780 path — static leaders drawn on load, `@page` sized to the laid-out sheet, exported via headless Chrome (`chrome --headless=new --disable-gpu --no-pdf-header-footer --virtual-time-budget=8000 --print-to-pdf=<out>.pdf <out>.html`). A single Duden-like sheet.
- **`--web`:** the NEW interactive layer over the same markup — hover/click a callout highlights its target (and vice-versa), leader lines reflow on resize, and the page is **theme-aware** (light/dark via `prefers-color-scheme` + a toggle). Self-contained (inlined CSS/JS, `data:` images) so it opens by double-click and is safe to embed in a Docusaurus MDX page (csl-guides) or a Systema lesson.

Both are one HTML document with a print/screen split (`@media print` vs screen), so the source
never forks. Legend + provenance footer (dictionary, entry loci, transliteration + accent
mapping, model tier+version) on every artifact.

## Phase 4 — Wire + deliver

- Home for generated tooling + assets: extend [`SanskritLexicography/EntryAnatomy/`](https://github.com/gasyoun/SanskritLexicography/tree/master/EntryAnatomy) (where H780 lives), not a new tree. New exemplars land beside `pwg-`/`mw-`/`cdsl-entry-anatomy`.
- **Publish safety:** a specimen embeds a facsimile scan — run [`/publish-safety-check`](https://github.com/gasyoun/claude-config/blob/main/commands/publish-safety-check.md) before anything goes public (scan-server rights); the Cologne scans are display-permitted, but check per asset.
- Register a new public specimen page in the repo README + the consumer surface (csl-guides "how to read an entry" page / Systema learner track). Provenance line with tier+exact version.

## Hard rules

- **Reuse the H780 callout/leader/`@page` engine** — the skill adds image-mode + interactive-web + generalized headword selection, nothing that already exists in `build_entry_anatomy.py` gets rewritten.
- **Never present an invented anatomy as fact** — a proposed callout set is marked "proposed, verify"; sigla and citations stay verbatim (they are the evidence).
- **One source, two outputs** — print PDF and interactive web come from the same HTML via a print/screen split; do not maintain two documents.
- Distinct from [`/entry-anatomy`](https://github.com/gasyoun/claude-config/blob/main/commands/entry-anatomy.md) (textual, tinted, per-headword generator) — reach for `/entry-specimen` only when the deliverable is the *visual, leader-line-annotated picture/sheet*.

_Dr. Mārcis Gasūns_
