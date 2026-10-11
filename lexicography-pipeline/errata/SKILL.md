---
name: "errata"
description: 'Maintain per-book errata lists in the SanskritGrammar repo (errata.yml pattern) — ingest a printed edition''s typo sheet, regenerate ERRATA.md, cross-reference CHANGELOG to mark fixes, refresh monthly. Triggers: paste errata list, "mark this typo fixed".'
---

# errata

Use this skill when the user asks to run the migrated source command `errata`.

## Command Template


# /errata

_Created: 11-07-2026 · Last updated: 06-09-2026 (H4201 darwin routes)

Errata management for digitized reprint editions. **Every book folder carries an `errata.yml`**
(seeded empty if it has no corrections yet) → a generated `ERRATA.md` — corrections to the text,
each tagged with **who found it** and **when it entered our list**. Distinct from the CHANGELOG
(which records changes to *our digital edition*); the two are linked in Phase 3.

Errata enter a book two ways — do **not** assume a printed errata sheet exists:

- **A. Printed errata sheet** — some editions ship their own opechatki/errata list (e.g. Knauer's
  1908 print + the 2011/2015/2023 sheets). Transcribe it (Phase 1A).
- **B. Edition diff** — a book with no printed sheet (e.g. Kochergina) accrues errata by
  **comparing versions of its own text file over time**: when the file changes, diff old vs new
  and each corrected reading becomes an erratum (Phase 1B). This is why every folder has an
  `errata.yml` up front — so future edits have a home to land in.

Execute without confirmation for ingest + regenerate. The commit/PR follows the repo's normal
handoff-scoped git rule.

## Data model — source + generate (never hand-edit the .md)

Mirrors the repo's `.docx → .mdx` pattern:

| File | Role |
|---|---|
| `<Book>/errata.yml` | **hand-edited source** — one entry per correction |
| `<Book>/ERRATA.md` | **generated** — do not edit; produced by the generator |
| `ERRATA.md` (root) | **generated index** — per-book counts, links |
| `scripts/build_errata.py` | the generator (`npm run errata` → all books; `python scripts/build_errata.py <Book>` → one) |

Each `errata.yml` entry:

```yaml
- page: 117            # printed page number
  line: "8 сн."        # line ref in source shorthand (see below)
  read: "gañgāyā"      # the CORRECT form (Russian errata: "читать …")
  instead: "gaṉgāyā"   # what the print shows (Russian errata: "вместо …")
  found_by: "Knauer 1908 print"   # edition/errata-sheet label OR a person's name
  date_added: "2026-07-06"        # ISO date it entered THIS list (not the reporter's date)
  fixed_in: null       # optional: version tag once corrected in our .docx/.mdx, e.g. v0.2.0
  note: null           # optional free remark
```

The generator **de-duplicates**: rows sharing `(page, line, read, instead)` merge into one,
unioning `found_by` (so an erratum on both the 1908 print sheet and the 2023 sheet becomes a
single row citing both). Different *script representations* of the same slip (Devanāgarī vs IAST)
are intentionally **not** merged — they carry different information; keep both and cross-note.

## Phase 1A — Ingest a printed errata sheet → errata.yml

When the user pastes an errata list, transcribe it **faithfully** into the book's `errata.yml`.
Parsing the Russian errata shorthand:

- `стр. N` → `page: N`
- `M сн.` = M-th line **from the bottom** (снизу) · `M св.` = from the **top** (сверху) → `line: "M сн."`
- `чит.` / `читать X` **вм.** / **вместо** `Y` → `read: X`, `instead: Y`
- The reporting sheet's heading (e.g. "Кнауэр, опечатки 2015", "бумажная книга 1908 г.") → the
  `found_by` label for every entry under it (`"Knauer errata 2015"`, `"Knauer 1908 print"`).
- `date_added` = **today** (the date it entered our list), regardless of when the sheet is dated.

Cleanups while transcribing:
- **Drop header/garbage rows** — e.g. a literal `стр. Строка Напечатано Следует читать` (a table
  header pasted in) is not an erratum; omit it.
- **Split multi-ref entries** — `стр. 54, 14, 12 сн., чит. A вм. B и C вм. D` is two errata
  (line 14 → A/B, line 12 → C/D); write two rows.
- **Preserve verbatim**, including forms that look wrong — reproduce what the sheet says and add a
  `note:` rather than "correcting" the correction.
- Don't pre-dedup by hand — enter each sheet's report as its own row and let the generator merge
  exact matches (this keeps `errata.yml` a faithful transcription of every source).

If the book has no `errata.yml` yet, create one with a `work:` line and the header comment block
(copy the shape from `KnauerFrazy_1908/errata.yml`). Every folder in the repo should already have
one — a new source added later needs an `errata.yml` too (even if `entries: []`).

## Phase 1B — Edition diff → errata.yml (books with no printed sheet)

For a book that never had a printed errata sheet (Kochergina, the Zaliznyak sketches, …), errata
come from **comparing two versions of the book's own text file**. When the source `.mdx` changes
(a reconversion that fixes a reading, a hand-correction, a newer edition committed over an older
one), surface the changes:

```sh
python3 scripts/build_errata.py diff <Book> <old-git-ref> [<new-git-ref>]
# e.g.  ... diff KocherginaUchebnik_1998 HEAD~1        (HEAD~1 vs working tree)
#       ... diff KocherginaUchebnik_1998 v0.1.0 v0.2.0 (two released editions)
```

This writes `<Book>/errata.candidates.yml` — every line-level text **replacement** between the two
editions, as `read` (new) / `instead` (old), tagged `found_by: edition diff <old>→<new>` and
today's date. It is **gitignored review scratch, not a deliverable**. Then:

1. **Curate** — most diff hunks are reformatting/reconversion noise, not corrections. Keep only
   genuine text fixes; discard the rest. (Pure insertions/deletions are already excluded — only
   `replace` hunks are emitted.)
2. **Enrich** — add `page`/`line` (the diff can't know them) where you can; they're optional.
3. **Fold** the kept entries into `errata.yml`, then delete `errata.candidates.yml`.

Use this whenever a book's file changes and you want the change recorded as errata — that is the
standing "compare editions over time" mechanism for every folder.

## Phase 2 — Regenerate

```sh
npm run errata            # regenerate every book's ERRATA.md + the root index
# or one book:  python3 scripts/build_errata.py <Book>
```

The generator sorts by page (then св. before сн., then line number), numbers rows, merges dups,
and writes the "N errata (M open · K fixed)" summary line. If a new book's `.mdx` is on the
Docusaurus site, consider adding its `ERRATA.md` to `docusaurus.config.mjs` / `sidebars.mjs` so
the errata renders alongside the book.

## Phase 3 — CHANGELOG cross-reference (the monthly pass)

The link between the two lists: **when a typo was actually fixed in our digital edition, mark it.**

1. Read `CHANGELOG.md`. For any released entry describing a text correction to a book (the
   generator also prints a reminder for changelog lines that mention a book + a fix-verb —
   "fixed" / "corrected" / "исправл"; mere errata-cataloguing lines are deliberately ignored),
   find the matching erratum in that book's `errata.yml`.
2. Set `fixed_in: v0.X.Y` (the version that shipped the fix) on that entry. Re-run Phase 2 — the
   row now renders `✓ fixed in v0.X.Y (date)` instead of `open`, and the open/fixed counts update.
3. Conversely, if this pass *applied* a correction to the source `.docx`/`.mdx` (not just logged
   an erratum), add a CHANGELOG `[Unreleased]` entry for it and then set `fixed_in` — keeping the
   two in sync. A changelog text-correction with no matching errata row, or an errata row fixed in
   the source but still `open`, is the drift this pass exists to catch.

Run this pass **monthly** (or whenever the CHANGELOG gains a text-correction entry). Pairs with
`/cut-release`: promote the changelog, then set `fixed_in` to the just-cut version.

## Phase 4 — Commit + PR

Stage with an explicit pathspec (avoid the shared-index race):

```sh
git add -- <Book>/errata.yml <Book>/ERRATA.md ERRATA.md scripts/build_errata.py package.json
git commit -m "errata(<Book>): <N> corrections from <sources>"
```

Open a PR and merge it per the repo's handoff-scoped git rule (no confirmation ask when the work
is scoped to this errata task). Update `.ai_state.md` if the errata backlog changes state.

_Dr. Mārcis Gasūns_
