---
name: "book-press-prep"
description: 'Camera-ready book/print prep — docx→mdx cleanup, errata.yml seeding with provenance, appendix assembly, house-template front/back matter — mechanics only, never rewrite author prose. Triggers: "print prep", "подготовка к печати". Do not use to rewrite author prose or compose chapters — that is /book-chapter-drain; this is mechanics-only camera-ready prep.'
---

# book-press-prep

Use this skill when the user asks to run the migrated source command `book-press-prep`.

## Command Template


# /book-press-prep <repo>/<book-dir> [--report-only]

_Created: 07-07-2026 · Last updated: 05-09-2026_

Distills the print-prep family: H246 (Fable 5 — Gasūns dhātu 2026 edition mechanics, SanskritGrammar [PR #29](https://github.com/gasyoun/SanskritGrammar/pull/29)), H275 (Fable 5 — its RuWritingStyles report-only style pass), H268 (Fable 5 — Sundarakāṇḍa «Литературные памятники» camera-ready, CommentaryStrategies). The `paper-*` skills are journal-shaped; this is book-shaped.

## The one guardrail that defines the skill

**Mechanics only — never rewrite the author's prose.** Verbatim (H275): "НЕ переписывать прозу в тексте книги — только отчёт с предложениями; финал за человеком-редактором." The agent fixes conversion artifacts, numbering, tables, references, apparatus; for style/content it produces **draft proposals and a ranked report**, and a human editor (or the author) finalizes. Corollaries: never invent numbers to fill a gap ("НЕ выдумывать цифры … только ссылки на существующие датасеты"); nothing loses a `review_required` flag without its human gate (H268); key statements (положения, Заключение, объект исследования) get *predloženie*-drafts, human-visaed.

## Phase 0 — Repo constraints first

Book dirs often live on a non-main branch with gitignore allow-lists — verify before writing (H246: `GasunsDhatu_2014/` exists only on `chore/errata-kochergina-waiting`; only `*.mdx` + `*_media/` pass the allow-list — new files must conform or be explicitly allow-listed; PRs target that branch, NOT main). Site-bearing repos require a green `npm run build` before PR. Watcher + concurrent-session checks as usual.

## Phase 1 — Conversion cleanup (mechanical)

- Strip Pandoc docx→mdx escapes (`\[ \]`, `\"`, literal `<u>`), fix OCR/typo strays, convert legacy media (`.wmf` → PNG/SVG).
- Reconstruct broken tables from the source `.docx`/`.pdf` as rst-tables — never approximate from the corrupted output.
- Reconcile numeric mismatches across the text to single values **with footnotes recording the variance**; fix section-numbering defects; add terminological notes where the human review demanded them.

## Phase 2 — Errata seed

Record every mechanical fix as an entry in `<book-dir>/errata.yml` per the `/errata` skill's schema (as used by the series' other books), each carrying `found_by: <H###-review, DD-MM-YYYY>` provenance. Fixes without an errata trail are invisible to the next edition.

## Phase 3 — Appendix + matter assembly

- Assemble appendices per the ruled plan: originals kept as-is; oversized apparatus (concordances) as **condensed extract + link to the digital crosswalk**, never re-typeset in full; add a dated "current-state" inset linking the live datasets that supersede the book's frozen tables.
- For series volumes (ЛП/Наука): **confirm the house template against real corpus models** (existing ЛП volumes) — "do not invent it". Apparatus exports emit print-formatted endnotes (a `build_book_apparatus`-style script), keeping distinct annotation strata separate — never inflate one stratum with another's notes.

## Phase 4 — Style pass (`--report-only` mode; this is H275)

Run the register-critique harness (RuWritingStyles «Совет филологов» for Russian prose) over the main text chapter-by-chapter + appendices. Output = a ranked **"список худших мест"** (раздел → абзац → диагноз → предложение правки) to `<book-dir>/revision-2026/RWS_REPORT.md`-style file (respect the allow-list; `.md` not published to the site). Focus where the human reviewers pointed. The agent does the pass and the list; **сплошную переработку прозы агент НЕ делает** — the human editor works from the report. Reference existing proposal drafts instead of duplicating them.

## Delivery

PR into the branch that carries the book dir; green build; errata + `.ai_state.md` + registry row updated; monograph-scale works registered in `ARTICLES.md` (Mxx). Human gates left as **named, small, final actions** in GTD — the acceptance shape (H268): "every machine-buildable part finished and every human gate reduced to a named, small, final action." Model tier + exact version stated.

_Dr. Mārcis Gasūns_
