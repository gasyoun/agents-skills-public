---
name: "rws-passport-build"
description: 'Author ONE RuWritingStyles style passport — extract register features from a corpus text, encode to house ClaudeStyles, validate via the rws council. Fable 5 (claude-fable-5), Sonnet 5 (claude-sonnet-5). Trigger: "построй паспорт".'
---

# rws-passport-build

Use this skill when the user asks to run the migrated source command `rws-passport-build`.

## Command Template


# /rws-passport-build

_Created: 24-08-2026 · Last updated: 02-09-2026_

The repeatable authoring path behind four passports built in one day (Бартольд 1927 «Киргизы» · Тураев 1920 · Крачковский 1925 Усама · Голенищев 1880 «Книга мёртвых» — [H3381](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H3381-OxAlpha_RuWritingStyles_bartold-passport-build_23.08.26.md), [H3382](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H3382-OxAlpha_RuWritingStyles_turaev-passport-build_23.08.26.md), [H3383](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H3383-OxAlpha_RuWritingStyles_krachkovskij-passport-build_23.08.26.md), [H3384](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H3384-OxAlpha_RuWritingStyles_golenishchev-passport-build_23.08.26.md), OxAlpha lane) and the nine Zalizniak-family passports before them ([RuWritingStyles/ClaudeStyles/](https://github.com/gasyoun/RuWritingStyles)). A passport is NOT an essay about an author — it is an operating persona that changes how `rws run` rewrites text.

## Phase 0 — Pick the grounding corpus text

1. One author = one era-defining text the author actually wrote, public-domain, digitized in the repo or ingestible under [license-gated-ingest](https://github.com/gasyoun/claude-config/blob/main/commands/license-gated-ingest.md). The passport is grounded in THAT text, not in the author's biography and not in secondary criticism.
2. Sanity-check volume: enough to quote every claim at least once (~a book chapter minimum); note genre (публицистика/перевод/научная проза) — the passport inherits the genre's register.

## Phase 1 — Extract register features FROM the text, not about the author

For each feature class below, the ONLY valid evidence is a verbatim quote from the corpus:

- синтаксис (средняя длина и строение предложения; парцелляция; порядок слов);
- лексика (любимые слова, архаизмы/ориентализмы/латинизмы, запретные пласты);
- тон и модальность (ирония, риторические вопросы, оценки);
- ритм и абзац (длина абзаца, связки, повторы-подхваты);
- обращение с фактами, именами, датами (как вводятся, как оговориваются);
- характерные зачины и концовки.

Anti-patterns section comes from what the author demonstrably does NOT do (absence claims need counter-evidence from the same corpus).

## Phase 2 — Encode to the house template

Write `ClaudeStyles/<name>-style.md` with EXACTLY the structural sections the existing passports share:

```
# Пользовательский стиль для Claude: <имя>-стиль
## Введение
## Основные установки
## Механика ответа
## Синтаксис и тона
## Лексика
## Обращение на вы
## Числа и примеры
## Что не делать
## Лингвистические отсылки
## Готовый текст для Claude
## Пример диалога в этом стиле
## Краткая шпаргалка
```

Rules of encoding: every prescriptive claim carries its corpus quote inline («вот строка из текста»); «Готовый текст для Claude» is self-contained (usable as a system-prompt block without reading the rest); «Краткая шпаргалка» is ≤10 bullets. **Inventing a quote is the named FAIL** — if the corpus does not support a feature, cut the feature.

## Phase 3 — Council gate before landing

Run the draft through [`/rws-council`](https://github.com/gasyoun/claude-config/blob/main/commands/rws-council.md) (or a blind rewrite probe: two passages, one by the passport, one authentic — can a reader tell which is which). The passport lands as a usable `rws run` persona only after the gate; a failing gate returns to Phase 1 with the specific misfires named.

## Phase 4 — Register and wire

Commit `ClaudeStyles/<name>-style.md` to [RuWritingStyles](https://github.com/gasyoun/RuWritingStyles); add the passport to the council rosters that should carry it ([`/rws-council`](https://github.com/gasyoun/claude-config/blob/main/commands/rws-council.md) general panel, orientalist panels per the [H944](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H944-Fable_RuWritingStyles_orient-leningrad-vostokoved-passports_14.07.26.md) Fable 5 lineage); one line in the repo's index/changelog.

## Hard rules

- No quote may be paraphrased into a "quotation" — verbatim or absent.
- Biography and criticism inform NOTHING except Phase 0 selection.
- RU output stays in the author's register; modern slang in a 1920s passport voice is a defect.
- One passport per pass; batch days do them serially (the observed cadence: ~4/day).

## Not this skill

| Situation | Use instead |
|---|---|
| Running existing passports as review panels | [`/rws-council`](https://github.com/gasyoun/claude-config/blob/main/commands/rws-council.md) · [`/zaliznyak-council`](https://github.com/gasyoun/claude-config/blob/main/commands/zaliznyak-council.md) |
| Style-of-record policy for a product domain | [`/register-style-guide`](https://github.com/gasyoun/claude-config/blob/main/commands/register-style-guide.md) |
| De-AI-ing a finished text with existing styles | [`/litredaktor`](https://github.com/gasyoun/claude-config/blob/main/skills/litredaktor/SKILL.md) |

_Dr. Mārcis Gasūns_
