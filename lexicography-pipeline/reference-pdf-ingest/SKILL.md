---
name: "reference-pdf-ingest"
description: 'Ingest ONE untracked inbound scholarly PDF — extract metadata, give it a self-identifying home, mint BibTeX, crosslink to its paper/topic, log in the reference index. Sonnet 5 (claude-sonnet-5). Triggers: "ingest this PDF", untracked .pdf.'
---

# reference-pdf-ingest

Use this skill when the user asks to run the migrated source command `reference-pdf-ingest`.

## Command Template


# /reference-pdf-ingest

_Created: 14-07-2026 · Last updated: 26-08-2026_

Turn a raw inbound PDF into a first-class reference: confirmed metadata, a
durable self-identifying filename, a BibTeX entry with a sane cite-key, a
crosslink to whatever it supports, and a row in the reference index. This is the
**intake** step that has no owner today — [`/crosslink-weave`](https://github.com/gasyoun/claude-config/blob/main/commands/crosslink-weave.md)
and [`/findings-append`](https://github.com/gasyoun/claude-config/blob/main/commands/findings-append.md) are downstream of it (they weave and
log an asset that already has a home; this skill gives it one).

This skill was minted for exactly the shape of `Sprache in Zahlen.pdf` and
`Stache-Weiske_Bö-MW.pdf` sitting untracked in SanskritLexicography's `papers/`
(since moved on by hand); whenever `git status` shows the next untracked `.pdf`,
this is its owner.

Argument: the path to the inbound PDF, or infer it from the conversation / from
`git status --porcelain` showing an untracked `.pdf`.

**Tier:** metadata extraction is mechanical, but confirming a title/author/venue
from a messy scan and judging which paper/topic it supports is judgment — Sonnet 5
(`claude-sonnet-5`).

## Phase 1 — Extract + confirm metadata

Never trust the filename alone. Read the actual document:

1. Read the first page (and the last, for a colophon/references) with the Read
   tool's PDF `pages` mode — `pages: "1"`, then `pages: "1-2"` if the title block
   spills. For a scanned image-only PDF the Read tool sees the page image; if text
   extraction is needed, `pdftotext -f 1 -l 2 <file> -` (poppler) or fall back to
   the embedded document properties (`pdfinfo <file>`).
2. Capture: **title**, **author(s)**, **year**, **venue/publisher**, and
   language. Note the producing tool / embedded title from `pdfinfo` as a
   cross-check (the existing [`REFERENCES.md`](https://github.com/gasyoun/SanskritLexicography/blob/master/REFERENCES.md)
   rows are built exactly this way — embedded metadata first, inferred fields
   marked).
3. **Mark every inferred field explicitly** (`*venue inferred from filename*`).
   A field you could not confirm from the document is a caveat, not a fact — carry
   the marking through to the index row, matching the existing REFERENCES.md
   convention.

## Phase 2 — Durable, self-identifying home

- **Where:** the repo's reference home. In SanskritLexicography that is
  [`papers/`](https://github.com/gasyoun/SanskritLexicography/blob/master/papers)
  (author-supporting publications) or the repo root for large HTML/PDF snapshots
  already indexed in REFERENCES.md. If the PDF is already inside `papers/`, it may
  only need renaming — do not move it out.
- **Filename:** self-identifying per the standing convention — ≥2–3 content tokens
  naming author + subject, ASCII-safe, then year. `Stache-Weiske_Bö-MW.pdf` is
  fine (author + subject); a bare `scan.pdf` / `paper2.pdf` / `download.pdf` must be
  renamed. Prefer `Author_ShortSubject_YYYY.pdf`. Use `git mv` if the file is
  already tracked; a plain rename if still untracked.
- Preserve the file's bytes — never re-encode or "optimize" a PDF on ingest.

## Phase 3 — BibTeX entry

1. **Find the bib file.** Look for one first —
   `find . -maxdepth 3 -iname '*.bib'`. SanskritLexicography currently has **none**;
   its reference system of record is the Markdown table in
   [`REFERENCES.md`](https://github.com/gasyoun/SanskritLexicography/blob/master/REFERENCES.md).
   If a `.bib` exists, append to it. If none exists, create
   `references.bib` at the repo root (a real, parseable BibTeX file that mirrors the
   REFERENCES.md index) and note it in REFERENCES.md so the two stay paired — do not
   silently invent a bib file in an odd location.
2. **Cite-key:** `authorlastnameYYYYkeyword` (lowercase, ASCII, no diacritics) —
   e.g. `stacheweiske2019boehtlingk`. Keep it collision-free against existing keys
   in the bib.
3. Pick the right entry type (`@article` / `@incollection` / `@book` /
   `@techreport` / `@misc`) and fill only confirmed fields; drop (don't guess)
   fields you could not verify. Add a `note = {ingested DD-MM-YYYY; venue inferred}`
   for anything marked inferred in Phase 1.

## Phase 4 — Crosslink to what it supports

Decide which paper (an `Axx`/`Mxx` in
[`Uprava/ARTICLES.md`](https://github.com/gasyoun/Uprava/blob/main/ARTICLES.md)) or
which topic doc this reference substantiates:

- If it backs a specific manuscript/section, add the cite-key (or a full blob-URL
  link to the PDF) into that doc, and hand the systematic weave to
  [`/crosslink-weave`](https://github.com/gasyoun/claude-config/blob/main/commands/crosslink-weave.md) if the target is a hub doc whose sections
  reference siblings only in prose.
- If it is a reusable structural finding (a survey, a comparison it enables), route
  it via [`/findings-append`](https://github.com/gasyoun/claude-config/blob/main/commands/findings-append.md) to the right FINDINGS hub.
- If it supports nothing yet, say so — an un-cited reference is still validly
  ingested; do not fabricate a link to the nearest tangential paper.

## Phase 5 — Log it in the reference index

Add one row to [`REFERENCES.md`](https://github.com/gasyoun/SanskritLexicography/blob/master/REFERENCES.md)
in its existing table shape: **File** (full blob URL link) · **What it is** · **Date**
· **Producer / format** · **Size**. Carry every *inferred* marking from Phase 1.
Bump the doc's `Last updated` header (the file dates convention: find the real
creation date via `git log --follow --diff-filter=A`, bump only "Last updated").

## Phase 6 — Deliver

- Commit the moved/renamed PDF + the bib change + the REFERENCES.md row together
  with model-tier provenance; PR-and-merge per the handoff-scoped autonomy rule if
  a handoff covers the work, else a normal branch→PR (SanskritLexicography is a
  shared-tree guarded repo — work in a worktree off `origin/master`, never commit on
  the main-tree checkout).
- Add a `[Unreleased]` bullet to [`CHANGELOG.md`](https://github.com/gasyoun/SanskritLexicography/blob/master/CHANGELOG.md)
  — a new tracked reference is a durable artifact.
- One-line report: the file's new path, its cite-key, and what it was crosslinked to.

## When NOT to use

- **The PDF is primary corpus / data**, not a reference — a dictionary scan to be
  digitized, a headword export, a corpus dump. That is a data-ingest (register it in
  the data census / `datasets.json`, see the org CLAUDE.md spine table), not a
  bibliography entry. This skill is for *supporting literature you cite*, not *data
  you process*.
- **A batch of many PDFs** — this skill is one-PDF-at-a-time by design (each needs
  metadata judgment); loop it, don't try to auto-classify a folder blind.
- **A born-digital reference with a DOI you already have** — just add the BibTeX and
  index row; you don't need the extract-from-scan phase.

## Safety notes

- Read the PDF before citing it — never publish or index a reference on filename
  faith; a mislabeled scan pollutes the bib permanently.
- Preserve inferred-field markings end to end; an unconfirmed venue asserted as fact
  is the exact defect REFERENCES.md's "*not verified*" convention guards against.
- Never strip or add a BOM to text files you touch; PDFs are copied byte-for-byte.

_Dr. Mārcis Gasūns_
