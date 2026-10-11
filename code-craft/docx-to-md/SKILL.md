---
name: "docx-to-md"
description: 'Batch .doc/.docx to .mdx beside source (LibreOffice+Pandoc): tables as GFM/grid in rst-table fences, images extracted, skips fresh outputs (--force). Default target SanskritGrammar. Triggers: "convert docx to mdx", "batch pandoc Word files".'

---

# docx-to-md

Use this skill when the user asks to run the migrated source command `docx-to-md`.

## Command Template


# /docx-to-md

_Created: 11-07-2026 · Last updated: 28-08-2026_

Batch-convert Word files → `.mdx`, always handling **both** `.doc` and `.docx`, without hurting
tables or formatting.

Pipeline, run in order over the whole tree:

1. **`.doc` → `.docx`** via **LibreOffice headless** (`soffice --convert-to docx`)
2. **`.docx` → `.mdx`** via **Pandoc** — pipe tables where GFM syntax fits, grid tables (wrapped in
   a plain ```rst-table fenced code block) everywhere else

**MDX everywhere is the default — not just csl-guides.** Earlier versions of this skill only
emitted `.mdx` + wrapped tables for a csl-guides target and fell back to plain `.md`/GFM elsewhere.
That fallback turned out to **actively corrupt data**: Pandoc's plain `markdown` writer doesn't
just use pipe/grid tables — by default it also emits **"simple tables"** (bare dash-underlined
columns, no `|`/`+` at all) whenever a table is simple enough. Neither simple tables nor grid
tables are valid GFM syntax, so on GitHub/any MDX site they render as **inert plain text** — for
[SanskritGrammar](https://github.com/gasyoun/SanskritGrammar) specifically (feeding book sites like
[buhler-sanskrit-book](https://github.com/gasyoun/buhler-sanskrit-book)), a scan of the six already-converted
`.md` files found 424–944 grid-table border lines EACH plus an unknown number of simple tables —
real declension/conjugation tables that were silently unrenderable. Fix, applied universally now:

1. **Disable `simple_tables`/`multiline_tables`** (`--to markdown-simple_tables-multiline_tables`)
   so Pandoc can only choose pipe tables or grid tables — verified this collapses 100% of a test
   file's simple tables into grid tables (`grep -c '^[- ]\+$'` → 0 after).
2. **Wrap every grid table** in a plain ```rst-table``` fenced code block — **no `:::` directive
   wrapper**. (An earlier version of this skill used a `:::table` container-directive design
   shelling out to Pandoc at Docusaurus build time; that was replaced 06-07-2026 after finding
   buhler-sanskrit-book already had a superior prior-art implementation — see below.)
3. Output extension is always `.mdx` (not `.md`) since rendering requires MDX's remark
   pipeline to be active on the consuming site.

**Rendering needs a matching remark plugin registered wherever the `.mdx` is actually built**
(a Docusaurus site's `remarkPlugins`). It does NOT need to exist in the source repo itself — most
`.mdx` produced by this skill (e.g. everything under SanskritGrammar) is raw source material later
hand-curated into a book site, not built in place. The plugin is a **pure-JS RST grid-table
parser** (`parseRstGridTable` + `buildTableAst`) that emits real `mdxJsxFlowElement` `<table>`/`<td
rowSpan=N>`/`<td colSpan=N>` nodes directly — no Pandoc dependency at build time, true merged-cell
`rowSpan`/`colSpan` preserved (mdast's own table-cell type has no such fields, so a
Pandoc-HTML-via-mdast route silently drops merges — this is why the plugin bypasses mdast tables
entirely). Two sites currently register it:
[buhler-sanskrit-book](https://github.com/gasyoun/buhler-sanskrit-book/blob/main/src/remark/rstTable.ts) (**canonical,
original implementation, TypeScript**) and
[csl-guides](https://github.com/sanskrit-lexicon/csl-guides/blob/main/src/remark/rstTable.mjs) (ported to plain JS,
since csl-guides has no TS toolchain). Keep both copies in sync by hand — there is no shared npm
package yet. Porting to a **new** future book site is a one-time setup cost, not a per-conversion
one — do it once per site, not once per file.

**Argument:** an optional target directory (default `~/Documents/GitHub\SanskritGrammar`)
and/or the flag `--force`. Examples:

- `/docx-to-md` — convert everything under SanskritGrammar, skipping up-to-date files.
- `/docx-to-md ~/Documents/GitHub\SomeRepo` — different target tree, same `.mdx`+`rst-table` behavior.
- `/docx-to-md --force` — regenerate every `.mdx` even if one already exists.

Run without confirmation. This only **creates `.mdx`, `.docx` (from `.doc`), and `*_media/` folders** —
it never edits or deletes a source file.

---

## Design decisions (locked)

| Decision | Value | Why |
|---|---|---|
| Legacy `.doc` | Convert to `.docx` first via LibreOffice | Pandoc cannot read binary `.doc`; LibreOffice is headless and needs no interactive desktop. |
| **Existing `.docx` is authoritative** | A `.doc` is **never** converted when a sibling `.docx` already exists | Protects hand-corrected Unicode `.docx` from being clobbered by a LibreOffice roundtrip of an old `.doc`. `--force` does NOT override this — delete the `.docx` by hand to force a fresh conversion. |
| Output location | **Beside** the source, same basename `.mdx` | Keeps each converted file next to its source in the same subfolder. |
| Images | **Extract** via `--extract-media` to `<basename>_media/` | Embedded images become real files with **relative** links — nothing lost, output stays portable. |
| MD flavor | Pandoc `markdown-simple_tables-multiline_tables` (pipe + grid tables ONLY), grid tables post-process wrapped in ```` ```rst-table ```` | Pandoc's default table-style auto-pick includes "simple tables" (bare dashes, no `\|`/`+`) which are **not valid GFM/MDX syntax** and render as inert text — disabling that extension forces every table into pipe (renders natively) or grid (wrapped) form. Universal across every target, not just csl-guides. |
| **MDX-safety pass** (`make_mdx_safe`) | After grid-wrapping, neutralize every Pandoc-ism MDX v3 rejects: `[x]{.underline}`→`<u>`, strip `{#id}`/`{width=}` attr blocks, escape literal `{ }`, strip raw `<p>`/`<td>`/`<colgroup>` block-HTML, drop degenerate `<2`-border rst-tables, unescape intended tags, `\<`/`\>`→`&lt;`/`&gt;` | Raw Pandoc `markdown` is **not** valid MDX — `{` parses as a JS expression, `<` as JSX, and `remarkRstTable` re-parses each table CELL through the same parser, so these crash `npm run build` in prose AND tables. Ported back from the SanskritGrammar book-site scaffold (06-07-2026) where these took all 6 grammar books green. |
| **Frontmatter** (`add_frontmatter`) | Prepend `title` + `sidebar_label` YAML when absent — known grammar sources by a filename lookup, else the first `# H1`, else the humanized filename | Docusaurus/MDX needs frontmatter: without it, gray-matter misreads a leading setext heading (`Title\n---`) as a frontmatter delimiter and the build dies. Also sets a clean page title/sidebar label. Skipped if the file already starts with `---`. |
| **Book-site tooling** (`site_tools.py`, Phase 4) | `fidelity` QA report · `images` WebP-optimize (path-scoped link rewrite) · `site` idempotent Docusaurus scaffold + plugin vendor + `npm build` with per-book `<table>` counts | The `.mdx` are meant to build in a Docusaurus site; these turn the manual QA + site-scaffold + build-verify (all done by hand 06-07-2026) into re-runnable commands. Optional — the converter itself doesn't need them; run when standing up or QA-ing a book site. |
| Re-run | **Skip** when the output is newer than its source | Idempotent batches; `--force` regenerates the output. |
| Wrapping / headings | `--wrap=none`, `--markdown-headings=atx` | One paragraph per line (clean diffs); `#`-style headings. |

Encoding is UTF-8 throughout, so Devanagari / IAST / Cyrillic / diacritics survive intact.

> **Why not Word COM for `.doc`?** Word's COM automation can create the app object but **cannot open
> documents in a non-interactive tool session** (fails with `0x800A16A6` regardless of PowerShell
> version, Mark-of-the-Web removal, or sandbox state). LibreOffice headless is the reliable path.

---

## Phase 1 — Preflight

```bash
pandoc --version | head -1
```

Locate LibreOffice (only needed if the tree contains `.doc` files):

```bash
ls "/c/Program Files/LibreOffice/program/soffice.exe" 2>/dev/null || \
ls "/c/Program Files (x86)/LibreOffice/program/soffice.exe" 2>/dev/null || echo "no soffice"
```

- **Pandoc missing** → stop; tell the user to install it (`choco install pandoc -y`), then re-run.
- **soffice missing AND the tree has `.doc` files** → install LibreOffice. It needs admin, so run
  elevated (pops a UAC prompt the user must approve):
  ```
  Start-Process -FilePath "choco" -ArgumentList "install libreoffice-fresh -y --no-progress" -Verb RunAs -Wait
  ```
  If the tree has no `.doc` files, skip this — LibreOffice is not required.

## Phase 2 — Run the committed converter

The converter is a committed, deterministic tool —
[`Uprava/tools/docx_to_mdx.py`](https://github.com/gasyoun/Uprava/blob/main/tools/docx_to_mdx.py)
(extracted verbatim from this skill's former ~320-line inline program —
H929 (Opus) — claude-config docx-converter tool extraction and handoff trim). Do **not**
re-emit it inline. Run it, passing through whatever argument/flag the user gave:

```bash
python ~/Documents/GitHub/Uprava/tools/docx_to_mdx.py [TARGET_DIR] [--force]
```

It walks the tree, skips Word lock files (`~$*.doc*`), converts each `.doc` → `.docx` via
LibreOffice (never overwriting an authoritative sibling `.docx`) and each `.docx` → `.mdx` via
Pandoc (`markdown-simple_tables-multiline_tables`, `--wrap=none`, `--markdown-headings=atx`,
`--extract-media`), then wraps every grid table in a plain `rst-table` fenced code block, runs the
full `make_mdx_safe` MDX v3 pass, and prepends YAML frontmatter. Idempotent — skips an `.mdx` newer
than its source unless `--force`; defaults to the SanskritGrammar checkout when no `TARGET_DIR` is
given. Every transform is spelled out in the **Design decisions** table above and in the tool's own
header + inline comments — change behaviour **in the tool**, never by re-pasting a fork here.

## Phase 3 — Verify fidelity

Don't trust exit codes alone — **actually build the consuming Docusaurus site if one exists**
(SanskritGrammar itself has one directly over the book folders; csl-guides and
buhler-sanskrit-book are the other two). A clean MDX compile is the only real proof; grepping the
`.mdx` text for suspicious patterns is a fast pre-check, not a substitute. Two real, previously
undetected bugs were only caught by actually running the build (06-07-2026) — check for both:

- **Bare-dash "simple table" leftovers** (`grep -cE '^[- ]+$' out.mdx`, should be 0) — sanity
  check that `-simple_tables-multiline_tables` took effect.
- **Stray escaped angle brackets** (`grep -c '\\<\|\\>' out.mdx` — count of literal `\<`/`\>`
  sequences, should be 0 after `escape_stray_angle_brackets` runs) — these crash the ENTIRE
  page's MDX compile with "Unexpected closing slash, expected an open tag first", and can hide
  inside a wrapped table cell's text too (the render plugin re-parses cell content as MDX). This
  is not a hypothetical: a single stray `\<` in one file broke that file's whole build, and a scan
  of all 6 SanskritGrammar sources found 200 instances across 4 of the 6 files.
- **If a consuming site exists, build it** and confirm both a real `<table><thead>` renders for at
  least one `rst-table` block, AND the build has zero MDX compile errors (Docusaurus reports them
  as `[ERROR] Client bundle compiled with errors therefore further build is impossible` — a
  green/warn-only build is required, not just "the process exited"). If converting into a repo
  with **no** consuming site yet, this step isn't possible — just confirm the fencing is
  well-formed (balanced ```` ``` ```` pairs, no stray blank lines splitting one table into two).
- If a `*_media/` folder was created, confirm the output references its images with **relative** links.
- Confirm scripts survived: no `�` replacement characters; Devanagari / Cyrillic / diacritics intact.

A quick script check:

```bash
python -c "t=open(r'PATH',encoding='utf-8').read(); print('replacement_chars=', t.count(chr(0xFFFD)), 'stray_angle_brackets=', t.count(chr(92)+'<')+t.count(chr(92)+'>'))"
```

Report the SUMMARY back, list files written, and flag any `rst-table` block that failed to parse
(fix by hand-adjusting the grid-table borders — deeply merged cells are the usual cause).

---

## Phase 4 — QA report, image optimization, and `--site` build (optional)

Beyond the converter, the book-site tooling is a committed, single-sourced script —
[`SanskritGrammar/scripts/site_tools.py`](https://github.com/gasyoun/SanskritGrammar/blob/main/scripts/site_tools.py)
(the canonical copy; run it from that repo's `scripts/` — do **not** re-emit it inline or fork a
copy here). Four commands, all safe to re-run:

- **`python scripts/site_tools.py fidelity [ROOT]`** — QA report over the `.mdx`: replacement chars,
  leftover bare-dash simple tables (outside fences), grid + pipe table counts, and a relative-image
  check. This is the automated form of Phase 3's manual greps — run it every time.
- **`python scripts/site_tools.py images [ROOT]`** — downscale (>1400px) + WebP-convert `*_media`
  rasters, rewriting the **owning** book's `.mdx` links (path-scoped to the book folder, NOT a
  bare-basename replace — `image1.png` recurs across books and a global replace cross-contaminates
  other books' links into missing files; that bug broke the build 06-07-2026). Needs Pillow
  (`pip install Pillow`); skips gracefully if absent. Saved ~516 KB on the 6 SanskritGrammar books.
- **`python scripts/site_tools.py site [ROOT]`** — the `--site` one-command flow: **idempotent
  Docusaurus scaffold** (only writes files that don't exist — `package.json` with the `webpack
  5.97.1` override, `docusaurus.config.mjs` registering `remarkRstTable`, `sidebars.mjs`,
  `.gitignore`; **vendors the 3 `rstTable*.mjs` plugin files** from a sibling
  csl-guides/buhler-sanskrit-book checkout), then `npm install` (if needed) + `npm run build`, and
  prints per-book rendered `<table>` counts. A green build with non-zero table counts is the real
  proof the grid tables render. webpack MUST be pinned to 5.97.1 — 5.108 breaks webpackbar's
  ProgressPlugin on Node 24.
- **`python scripts/site_tools.py links [ROOT]`** — print each `.mdx` as its deployed Docusaurus
  route, read from `docusaurus.config.mjs` (`url` + `baseUrl` + docs `routeBasePath`); `md` or `url`
  form. (Absent from this skill's former inline copy — a live example of the copy-drift that
  extracting to one committed file removes.)

---


## Notes

- **Never edit or delete a source `.doc`/`.docx`.** Output is `.mdx`, derived `.docx`, and `*_media/` only.
- A pre-existing `.docx` beside a `.doc` is **authoritative** and left untouched — this prevented a
  hand-made `Buhler_Unicode.docx` from being clobbered by a LibreOffice roundtrip of an old `.doc`.
- **`.mdx` + wrapped grid tables is universal, not csl-guides-specific — this changed 06-07-2026.**
  The earlier per-repo branch (csl-guides → `.mdx`, everywhere else → plain `.md`/GFM) was found to
  silently corrupt data: Pandoc's plain `markdown`/`gfm` writers can emit "simple tables" (bare
  dashes, no `\|`/`+`), which aren't valid GFM/MDX and render as inert text — confirmed on
  SanskritGrammar's already-converted files (424–944 grid-table border lines *each*, plus an
  unmeasured count of simple tables, none of it renderable). **Rendering** still requires a matching
  `remarkRstTable` plugin on the consuming site (pure JS, no Pandoc dependency at that site's build
  time — see below) — but the `.mdx` *output format* from this skill no longer depends on the
  target repo having one already. Porting the plugin to a new book site is a one-time setup, done
  once per site as it comes online, not gated per conversion.
- **The design changed AGAIN same day, 06-07-2026: no `:::` directive wrapper.** The first
  implementation used a `:::table` container-directive shelling out to Pandoc (`--from rst --to
  html5`) at Docusaurus build time. That was replaced after finding buhler-sanskrit-book already
  had a superior prior-art implementation: a **pure-JS** `remarkRstTable` plugin
  (`rstTableParser.mjs` parses the grid-table text directly; `rstTableAst.mjs` builds real
  `mdxJsxFlowElement` `<table>`/`<td rowSpan=N>`/`<td colSpan=N>` nodes) that needs **no Pandoc at
  Docusaurus build time at all** and preserves true `rowSpan`/`colSpan` (an HTML round-trip through
  mdast tables can't carry those — mdast's table-cell type has no span fields). Current convention:
  a **plain** ```` ```rst-table ```` fenced code block, no `:::` wrapper.
- **Critical fix, same investigation: escape stray `\<`/`\>` before writing the output.** Pandoc
  escapes any literal `<`/`>` in the source prose (comparative-linguistics notation like `a < ā`,
  or garbage like a literal `<TBODY>` pasted into the original Word doc) as `\<`/`\>`. MDX's JSX
  tokenizer does not reliably respect that markdown escape and can crash the **entire page's**
  compile — and because `remarkRstTable` re-parses each table cell's text through the same
  MDX-aware parser, this bites inside tables too. Always run `escape_stray_angle_brackets` (replace
  `\<`→`&lt;`, `\>`→`&gt;`) on the Pandoc output before writing the `.mdx`. Found the hard way:
  1 instance broke ZalizniakKonspekt_2004's entire build; a scan of all 6 SanskritGrammar sources
  found 200 total instances across 4 of the 6 files, silently present since an earlier conversion
  pass that was never build-verified.
- Ad-hoc single-file conversion: `pandoc "in.docx" --from docx --to markdown-simple_tables-multiline_tables --wrap=none --extract-media "in_media" -o "in.mdx"`, then wrap any `+---+` grid-table block in a plain ```rst-table``` fenced code block, then replace any `\<`/`\>` with `&lt;`/`&gt;`.
  - `.doc` first → `soffice --headless --convert-to docx --outdir . "in.doc"`

_Dr. Mārcis Gasūns_
