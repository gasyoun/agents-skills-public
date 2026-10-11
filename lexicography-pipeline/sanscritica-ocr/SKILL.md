---
name: "sanscritica-ocr"
description: OCR scholarly scans into faithful Markdown — Cologne dictionary front matter (CDSL csldoc scans: title pages, prefaces, abbreviations, addenda) and general Russian/Latin-script bodies with Sanskrit in Latin (IAST), incl. pre-reform Cyrillic. Vision OCR — Fable 5 (claude-fable-5) tier, Opus 4.8 (claude-opus-4-8) fallback; DashScope qwen3-vl-plus lane for whole bodies (Uprava tools/ocr_vlm). Arg = dict code or all. Do not use for dictionary body-text corrections — that is /cologne-text-correction-pr or the csl-orig correction queue.

---

# sanscritica-ocr

Use this skill when the user asks to run the migrated source command `sanscritica-ocr`.

## Command Template


# /sanscritica-ocr

_Created: 03-07-2026 · Last updated: 01-10-2026_

OCR and translate the **front matter** (Vorspann: title pages, forewords/prefaces, abbreviation lists, addenda/corrigenda) of a Cologne Digital Sanskrit Lexicon dictionary, starting from the csldoc scan pages.

Index of all dictionaries with front matter:
`https://sanskrit-lexicon.uni-koeln.de/scans/csldev/csldoc/build/dictionaries/index.html`

Argument: a dictionary **code** (`PWG`, `MW`, `AP90`, `BUR`, `SKD`, …) or `all` to sweep every dictionary that has a preface index. Lower/upper case both fine; the URL code is lowercase.

This was first built for **PWG** (German), producing `PWG/prefaces/` with 27 pages × 3 languages + consolidated files. Reuse that as the reference output.

**The skill was named `cologne-preface-ocr` until 24-09-2026 (MG: «дурное название, давай просто sanscritica-ocr») — history and changelog keep the old name.** This command *is* the SanOCR workflow for CDSL front matter + the generalized RU/IAST scholarly-scan lane (see Phase 3 cross-checks). Codex twin: `~/.codex/skills/sanscritica-ocr/`. Public docs in `csl-guides`: finished editions → [`docs/dictionaries/ocr-prefaces.mdx`](https://github.com/sanskrit-lexicon/csl-guides/blob/main/docs/dictionaries/ocr-prefaces.mdx); **operator manual + Claude Code vs Grok attribution** → [`docs/dictionaries/preface-ocr-pipeline.mdx`](https://github.com/sanskrit-lexicon/csl-guides/blob/main/docs/dictionaries/preface-ocr-pipeline.mdx).

---

## Two engines (A authors; B audits or bulk-drafts)

| Engine | Role in this skill | When | Quality |
|---|---|---|---|
| **A — Vision band OCR** | **Author** of canonical `<code>prefNN.md` (Phase 3) | Cologne **csldoc PNG scans** (default) | Scholarly; Devanāgarī + 19th-c. orthography |
| **B — Tesseract crop-then-OCR** | **Audit layer only** on CDSL (Phase 3.5); optional primary draft on PD-style PDFs | After A drafts exist; or local multi-column PDFs | Fast; residual char noise — never auto-overwrite A |


---


---

## Autonomy rules

- Execute all phases without per-page confirmation. Vision OCR, file writes, `curl`, Python, and (when asked) git commits/pushes are pre-approved.
- Every Python script: `sys.stdout.reconfigure(encoding='utf-8')` and `sys.stderr.reconfigure(encoding='utf-8')`.
- Write files UTF-8 **without BOM** (`open(f,'w',encoding='utf-8')`, not `utf-8-sig`).
- Commit only when the user asks. Never commit temporary crop images (see Phase 7 — this is the easiest mistake to make).

---

## Phase 0 — Resolve target

```
CODE=<arg lowercased>           # e.g. pwg, mw, ap90 ; PW dict's code is "pw"
BASE=https://sanskrit-lexicon.uni-koeln.de/scans/csldev/csldoc/build
INDEX=$BASE/dictionaries/index.html
IMG_BASE=$BASE/_images
```

Decide the **output repo / directory**. Many dict repos are cloned under `~/Documents/GitHub/`, but the repo name is not always the dict code:


Resolve by: exact repo match → repo whose `CLAUDE.md`/`readme` names that dictionary → else create a working dir `GitHub/prefaces_<code>/`. Output goes in `<repo>/prefaces/`.

For `all`: enumerate dictionary codes from the index page (the `href="<code>.html"` links), then run Phases 1–7 per code, skipping any with no preface index (Phase 1 returns nothing).

---

## Phase 1 — Discover the scan pages

The front-matter index for a dictionary is `dictionaries/prefaces/<code>pref.html`. It is a Sphinx toctree linking sub-pages `<code>pref/<code>prefNN.html`; **each sub-page embeds exactly one scan** `<img src="../../../_images/<file>.png">`.

```sh
# the section list (toctree labels = "Title, vol. 1", "Foreword, 1", "Abbreviations, 1", ...)
curl -s "$BASE/dictionaries/prefaces/${CODE}pref.html" | grep -oE 'href="[^"]+\.html"[^<]*</a>'
# for each sub-page NN, the embedded scan filename:
curl -s "$BASE/dictionaries/prefaces/${CODE}pref/${CODE}prefNN.html" | grep -oE '_images/[^"]+\.png'
```

Build a manifest of `(NN, section_label, scan_filename, sub_page_url)`. Note the section labels and which volume each belongs to (title pages reset the volume). **If `<code>pref.html` 404s or has no sub-pages, this dictionary has no front matter — skip it** (record that for `all`).

⚠️ The toctree order is the authoritative page order, and it is occasionally not the lexical order of the image filenames (PWG had `--06`/`--07` swapped between "Foreword 5" and "Abbreviations 1"). Trust the toctree, not the filename sort.

---

## Phase 2 — Download scans

```sh
mkdir -p <repo>/prefaces/scans
# for each scan in the manifest:
curl -s -o "<repo>/prefaces/scans/<file>.png" "$IMG_BASE/<file>.png"
```

---

## Phase 3 — OCR each page (the critical phase)

**Output**: one Markdown file per page `<code>prefNN.md` in `<repo>/prefaces/`, with this header then the transcription:

```yaml
---
source_scan: <file>.png
source_page: <section label, e.g. "Foreword, 1">
volume: <n>
source_url: <sub_page_url>
---

# <heading as printed, e.g. VORWORT.>
```

### THE non-negotiable OCR technique

The scans are 3000–6800 px. **The Read tool downsamples a whole page below legibility** — reading the full PNG yields confident-looking but *wrong* text (this is the single biggest failure mode). You MUST crop to native resolution first:

1. Detect single- vs **two-column** (most prefaces/abbreviation lists are two-column; title pages are single, centered).
2. With PIL, split a two-column page into LEFT and RIGHT columns, then cut each column into 3–6 **overlapping horizontal bands**. Find the true column boundaries — text clipped at the right edge means the column is wider than your crop (re-crop wider).
3. **Hard limit: every crop you Read must be ≤ ~1900 px on its longest side**, or the request is rejected ("image exceeds dimension limit"). Do **not** upscale a half-page 2× (that blows past 2000 px). To read tiny text, crop a *smaller native-resolution region*, don't enlarge.
4. Read LEFT bands top→bottom, then RIGHT bands. For stubborn Devanāgarī conjuncts, crop that word at 3× into its own ≤1900 px tile.
5. Delete the temp crops when done (Phase 7).

Reference crop helper:
```python
from PIL import Image
im = Image.open(png)                       # e.g. 3304 x 4673
cols = {'L':(150,1730),'R':(1700,3180)}    # adjust to real column edges
ys = [300, 1700, 3100, 4520]               # band boundaries (trim margins)
for cn,(x0,x1) in cols.items():
    for i in range(len(ys)-1):
        y0 = ys[i]-(60 if i else 0); y1 = ys[i+1]+60
        im.crop((x0,y0,x1,y1)).save(f'_crops/{cn}{i}.png')   # each ≤1900 px
```

### Transcription rules

- Transcribe the **source language faithfully in its original orthography** — do not modernize. Cologne dict prefaces are variously German (PWG, PW, SCH, CCS, GRA), English (MW, AP, AP90, MWE, MD, WIL, …), French (BUR, STC), Latin (BOP), or Sanskrit (SKD, VCP, ABCH, KRM). Detect from the scan. Preserve 19th-c. forms (German *Theil, Litteratur, dass, Maassstabe*; long ſ as s).
- **Sanskrit stays verbatim**: Devanāgarī in Unicode when confidently legible (else `[?]`); Roman transliteration with full diacritics (ā ī ū ṛ ṝ ḷ ṅ ñ ṭ ḍ ṇ ś ṣ ḥ ṃ); editorial romanization (ç, ǵ, â) as printed. Personal names and work titles verbatim.
- Reflow prose into paragraphs (one blank line between); keep title-page line breaks; preserve footnotes, signatures, places, dates, printed page numbers (`[page V]`).
- **Abbreviation-list pages** are `key = source-title` lists — render one entry per line preserving the pairing; keep keys/titles verbatim. **Addenda/corrigenda** are `headword + gloss + reference` entries — keep headwords/Devanāgarī/references verbatim.
- Mark unreadable `[illegible]`, uncertain `[?]`. **Never invent plausible text** — fidelity over completeness.
- **Omit the digitizer stamps**: the tiny running header (`Otto Böhtlingk & … : Sanskrit-Wörterbuch …`) and footer (`Institute of Indology & Tamil Studies, Cologne University … <date>`) are added to every scan and are *not* part of the original.


### Parallelize

Fan out subagents (`general-purpose`), each assigned a few pages with the **full cropping recipe and rules above embedded in the prompt** (subagents do not see this skill). Group light pages (titles) together; give dense forewords ≤2 pages each; give heavy addenda one each. Subagents only read text/images — but if one dies on an image-dimension/500 error, it ignored the ≤1900 px rule: redo that page yourself with proper crops. **Verify quality**: an agent that did few tool calls and flagged "faint/uncertain" probably skipped cropping — re-OCR those pages.

---

## Phase 3.5 — Engine B audit layer (Grok / Tesseract path)

**Purpose:** use the cheap independent channel (Tesseract crop-then-OCR, same family as PD `run_ocr.py` v2 / `feat/ocr-v2-pipeline`) as a **second-opinion check** on Engine A drafts — not as a replacement transcription.

**When:** after Phase 3 has produced (or refreshed) `<code>prefNN.md` pages. Run before Phase 4 when feasible so re-vision fixes land before translation cost. Optional but **recommended** for new dicts and for any page already flagged uncertain in Phase 3. Skip only if Tesseract/`deu|eng|san` is unavailable and note that in run notes.

**Hard rules:**

1. **A stays gold.** Never promote B text into `<code>prefNN.md` unless a human or a **re-run of Phase 3 vision** on the flagged band confirms the change.
2. **Flags only.** B may disagree and still be wrong (PWG bake-off: title names, abbreviation keys, diacritics). Use B for **coverage / layout / omission** signals, not character truth.
3. **Ignore digitizer stamps** in B (`Institute of Indology…`, Cologne running headers) — A correctly omits them; B-only stamp text is not a skip flag.


---

## Phase 4 — Translations (English + Russian)

For every page produce `<code>prefNN.en.md` and `<code>prefNN.ru.md` (separate per-language files). Header adds `language: en|ru` and `translation_of: <code>prefNN.md`.

- If the **source language is English**, skip `.en.md` (the base `.md` is already English) and note it in the README; always produce `.ru.md`.
- Translate the source prose faithfully in scholarly register. **Leave verbatim**: Devanāgarī/Sanskrit (any script), work titles, bibliographic abbreviation keys, reference numbers, and embedded foreign quotations (e.g. an English quotation inside German stays English; add a parenthetical RU rendering of surrounding prose only).
- Headings: `# VORWORT.`→ `# Foreword` / `# Предисловие`; `# Title…`→ localized.
- Translation agents are text-only (no image limits) — safe to fan out in larger groups.
- Then run one **consistency proofread** pass: normalize RU names, catch OCR typos in the source `.md`, and fix any EN↔RU meaning divergence by checking the source.

---

## Phase 5 — Consolidated single-file editions

Produce one file per language with **all pages in order** + a table of contents: `<code>pref_all.de.md` (use the real source-language tag: `.de`/`.en`/`.fr`/`.la`/`.sa`), `<code>pref_all.en.md`, `<code>pref_all.ru.md`. Generate with a script committed as `<repo>/prefaces/build_combined.py` so it is reproducible. Generic, data-driven version (reads each page's YAML — no hardcoded page list):

Run `DICT=<code> python build_combined.py`. Sanity-check: H2 count == 1 (TOC) + number of pages; any extra H2 means a page body has a stray `#`/`##` (fix the source page's heading level — that happened with PWG's addenda page, which used `#` for a sub-section that should be `##`).

---

## Phase 6 — README index + cost note

**6a. In-folder index `<repo>/prefaces/README.md`.** What this is (dictionary, editors, years, source URL), the file-suffix conventions, a **Consolidated editions** table linking the `*_all.*` files and `build_combined.py`, and a **Contents** table — one row per page with Section / Vol. / source / en / ru links. Note source language, signatures/dates found, and that digitizer stamps were omitted. Use clickable Markdown links throughout (chat AND README; full blob URLs when referring on GitHub).

**6b. Repo-root index entry (the "preface index").** Also surface the new `prefaces/` folder from the dictionary repo's **root `README.md`** so the work is discoverable — this is what "add to the preface index" means (PWK/MWS precedent). Add: (1) a row to the repo's Contents/architecture table — `` | `prefaces/` | Front-matter OCR … — see Front matter below | ``; (2) a short **`## Front matter (`prefaces/`)`** section — what it is, the Cologne source URL, file conventions, links to the consolidated `*_all.*` editions and the in-folder `prefaces/README.md`, signatures/dates found, and any table/glyph notes; (3) a Timeline/changelog row (e.g. `| Jun 2026 | Front-matter OCR + EN/RU translation of the prefaces (`prefaces/`) |`). (The Cologne *online* index at `dictionaries/index.html` already lists any dict whose scans exist and is regenerated server-side — not editable from here; the repo README is the index you maintain.)


**6d. Surface in csl-guides.** After a dict's `prefaces/` is complete, update [`csl-guides/docs/dictionaries/ocr-prefaces.mdx`](https://github.com/sanskrit-lexicon/csl-guides/blob/main/docs/dictionaries/ocr-prefaces.mdx) **Available now** table (row + short "What's covered" bullet) **and** the attribution table in [`preface-ocr-pipeline.mdx`](https://github.com/sanskrit-lexicon/csl-guides/blob/main/docs/dictionaries/preface-ocr-pipeline.mdx) (Claude Code vs Grok). Do not leave the public index lagging local work.

---


## Phase 7 — Cleanup, commit, push

**Cleanup first (do not skip):** delete every temporary crop dir/file before staging. Subagents scatter them (`_crops/`, `_tmp/`, `tmp_crops/`, loose `_*.png`, `scans/_tmp/`). Also delete Phase 3.5 raw dumps `prefaces/_audit_b/` unless the user asked to keep them. Only these should remain tracked: `<code>prefNN.md|.en.md|.ru.md`, `<code>pref_all.*`, `build_combined.py`, `README.md`, `scans/<dict>*.png`, and (if non-trivial) `AUDIT_A_VS_B.md`.

```sh
rm -rf <repo>/prefaces/**/_crops <repo>/prefaces/_tmp* <repo>/prefaces/tmp_crops <repo>/prefaces/scans/_tmp <repo>/prefaces/_audit_b
rm -f  <repo>/prefaces/_*.png
git -C <repo> status --short prefaces        # eyeball: nothing but the keepers
```

Commit only when the user asks. If on the default branch, the user's PWG precedent committed straight to `master`; otherwise branch first. Stage `prefaces/` **and the root `README.md`** (the Phase 6b index entry + cost note), commit with a message describing scope, and end with:
```
Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com>
```
Push only if asked. If push is rejected (remote moved), `git fetch` + `git rebase origin/<branch>` (these commits only add new files → conflict-free) then push. If the working tree has an unrelated change blocking a rebase, do NOT `git stash` it (stash in a shared/linked tree can swallow another session's work — H859 class); commit it to a temp branch or work from a separate worktree instead.



---

## Phase 8 — Post-OCR enrichment (vote U18, H1617)

OCR Phases 0–7 produce faithful `prefNN` + translations. **Most sessions after OCR are enrichment, not re-OCR** — 17 handoffs in one week (H1501, H1530, H1543, H1558–H1561, H1569, H1571, H1580, H1591–H1597). Do **not** re-derive this ladder each time.

Run after Phases 0–7 for a dict (or over an already-Done dict). One session may own **one** rung; never skip the earlier rung's committed artifact.

| Rung | What | Evidence |
|---|---|---|
| **8a Abbr × body cross-check** | Keys from pref abbreviation lists vs body `.txt` / tagged `<ab>`; pilot then `--all` rollup | H1530, H1543 |
| **8b pref_only decompose** | Classify keys present only in pref (not body) — top-N + rollup note | H1560 |
| **8c Body naming authority** | Apply naming/authority decisions for pref↔body alignment | H1569 |
| **8d Align all dicts** | Scale alignment / rewrite-boost across dicts with path map | H1571, H1580 |
| **8e Legend store + fold registry** | Emit legend store, fold-table registry for UI/consumers | H1591, H1592 |
| **8f Abbreviations UI** | Rewrite public abbreviations surface (csl-guides) | H1593 |
| **8g Methods + citation** | Methods note + `CITATION.cff` / cite package for pref edition | H1558, H1594–H1597 |
| **8h Quality probes** | Bounded OCR/EN sample error rates; spot-check enrichment | H1559, H1596, H1597 |
| **8i Review packet** | Human review packet (Andhrabharati-style) when a third source exists | H1561 |


### Phase 8 rules

1. **Never overwrite Phase 3 gold** with automatic body/UI rewrites. Enrichment is additive TSV/MD/reports + UI consumers.
2. Prefer committed scripts under `csl-guides` / dict `prefaces/` (`pref_abbr_crosscheck.py`, legend emitters) — extend, don't fork.
3. Track issue comments on the owning tracking issue (e.g. csl-guides#123, PWG#210) with a results table when counts move.
4. BOM-free UTF-8; worktrees for multi-session dict repos.
5. If the rung is pure correction of body markup (`&c.` → `<ab>`), route via `/cologne-correction-queue` + monthly batch — not free edits to csl-orig.

---

_Dr. Mārcis Gasūns_
## References

Detail moved verbatim (read on demand — lives in the rendered skill copy's references/):

- `references/phase35-audit.md` — Engine B audit layer mechanics — feat/ocr-v2-pipeline branch, what to run, metrics table, AUDIT_A_VS_B.md shape, flag signals, after-audit steps
- `references/inventory-census.md` — Dictionary inventory census 23-07-2026 — Done precedents table, P0–P3 queue, dict-code→repo map, deleted-worktree note
- `references/ocr-vlm-lane.md` — Measured cross-checks & prompt rules (24-09-2026) — qwen3-vl-plus DashScope lane, diacritic anchors, pre-reform Cyrillic, traps
- `references/ru-names.md` — RU personal-name Cyrillicization reference list (Böhtlingk=Бётлингк …)
- `references/build-combined.md` — build_combined.py — the generic data-driven consolidated-edition generator
- `references/run-notes-cost.md` — 6c cost & run-notes — collapsible details block template and harness-telemetry tally rules
- `references/documentation-rule.md` — Tracking-issue results rule (23-07-2026) — results comment in the same pass, persist-tables contract
- `references/phase8-lineage.md` — Phase 8 enrichment lineage — the 17 Sonnet-5 handoffs behind rungs 8a–8i
- `references/gotchas.md` — Gotchas learned on PWG + later runs — crop limits, toctree order, temp crops, RU naming, MWE/WIL/PWK repo map
