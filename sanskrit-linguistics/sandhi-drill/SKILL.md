---
name: "sandhi-drill"
description: 'Generate sandhi practice - drill items (split/join/identify-the-rule) with same-class distractors from a text/lesson/rule-class; Anki .apkg+TSV, web quiz, or raw JSON out. Triggers: "sandhi-drill", "make sandhi flashcards". Sonnet 5 (claude-sonnet-5) tier. Do not use to lint or split real authored text — this only generates practice/quiz material; use /sandhi-check or /sandhi-split for actual text.'

---

# sandhi-drill

Use this skill when the user asks to run the migrated source command `sandhi-drill`.

## Command Template


# /sandhi-drill <text | lesson N | rule-class | rule>

_Created: 14-07-2026 · Last updated: 05-09-2026_

Scoped pedagogy generator over the sandhi programme data
([SANDHI_PROGRAMME.md](https://github.com/gasyoun/kosha/blob/main/SANDHI_PROGRAMME.md)):
the corpus rules carry attested `word+word` examples, and `sandhi_curriculum.tsv` gives the
lesson ordering. Reuse the csl-guides `sandhi-quiz.json` shape.

## Procedure

1. **Scope the item set.** From the argument: a text (its `<id>_sandhi.tsv`), a curriculum lesson (rules in that lesson), a class (visarga/…), or a single rule. Pull the attested examples.
2. **Build items** — three types: **split** (`nāgnir → na + agniḥ`), **join** (`na + agniḥ → ?`), **identify-rule** (`nāgnir → a a → ā`). Distractors = other rules of the SAME class ranked by frequency (plausible, not random). Tag each with its rule, class, and lesson.
3. **Emit the requested formats** (ask if unspecified): `sandhi_drills.json` (items), `sandhi_drills.apkg` + `.tsv` (Anki, via `genanki`; TSV fallback if absent), a self-contained theme-aware web quiz page.
4. Order by curriculum lesson (easy/high-frequency first); register a `sandhi-drills` dataset if it's a durable set.

**Non-sandhi drills** (vocab page, Zaliznyak declension, semdom thematic) → `/reader-layer` **drill surface** mode ([H1460](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1460-Sonnet_kosha_vocab-drills-inbrowser-page_22.07.26.md)–[H1462](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1462-Sonnet_kosha_semdom-thematic-vocab-axis_22.07.26.md), Sonnet 5 `claude-sonnet-5`), not this skill.

**Guardrail:** distractors must be genuinely of the same class (a plausible wrong answer), never a nonsense string — the drill teaches nothing if the wrong options are obviously wrong.

_Dr. Mārcis Gasūns_
