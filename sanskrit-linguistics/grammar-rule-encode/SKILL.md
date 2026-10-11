---
name: "grammar-rule-encode"
description: 'Encode a grammarian''s prose rules into a machine-readable rule table, validate predictions against an attested corpus - interpretive calls record the wording + rejected reading; per-rule accuracy reported. Triggers: "encode Whitney/Zaliznyak/Pāṇini rules".'

---

# grammar-rule-encode

Use this skill when the user asks to run the migrated source command `grammar-rule-encode`.

## Command Template


# /grammar-rule-encode <source> <corpus>

_Created: 07-07-2026 · Last updated: 26-08-2026_

Distills the prose→formal encoding discipline from H018 (Fable 5 — Whitney accent-axis table), H053 (Opus 4.8 — Zaliznyak nominal inflection index), H063 (Sonnet 5 — accent validation) and H115 (Sonnet 5 — the saṁyañc exception). One session = one rule table + corpus-validation report, PR-delivered. Judgment tier (a grammarian's silences need reasoned, documented calls) — Opus 4.8 (`claude-opus-4-8`)/Fable 5 (`claude-fable-5`).

## Phase 0 — Ground truth

- `/prior-art`: does a rule table for this source/axis already exist (repo `data/` tree, sibling repos, sanskrit-util)? Extend, don't fork.
- Read the actual source text (scan/OCR/edition) for every section being encoded — never encode from memory of what Whitney/Zaliznyak "says".
- **Hard guardrail (standing, from the Whitney I/VI memory): never write class/accent data without a grammar-or-Zaliznyak source.** Every cell in the output table traces to a cited section (`§`-level), or it does not exist.

## Phase 1 — The rule table

Machine-readable (TSV/CSV/YAML in the repo's convention), one row per rule, with mandatory columns:

- `rule_id` · `source_ref` (§/page of the edition) · formal condition · formal prediction.
- **`source_wording`** — the grammarian's actual words, quoted, for every row that required interpretation.
- **`reading_note` + `rejected_alternative`** — where the prose admits more than one formalization, record the reading taken AND the alternative rejected, with the reason. An interpretive call with no recorded alternative is a defect: it hides that a call was made at all (the H115 saṁyañc lesson — exceptions live in asides a naive encoding misses).
- `status` — `encoded` | `exception` | `@DECIDE` (see Phase 3).

## Phase 2 — Corpus validation

Validate rule predictions against an attested corpus (VedaWeb-style annotated corpus, DCS, or the repo's own attested-forms layer):

- Per-rule accuracy: n attested cases matched / contradicted / not-attested, in a committed table (log-tables convention: dated, provenanced, in the report — never chat-only).
- Contradictions are triaged, not averaged away: corpus annotation error vs rule mis-encoding vs a genuine exception the source states elsewhere vs a genuine gap in the source. Each contradiction row gets one of those four labels.
- A rule with zero attestations is reported as *untested*, never counted as validated.

## Phase 3 — Silences → @DECIDE

Where the source is genuinely silent (a paradigm cell no rule covers, a conflict between two sections), the row is parked as `@DECIDE` with the options laid out — **never silently resolved by analogy**, however plausible the analogy. Mirror the `@DECIDE` rows to `Uprava/GTD_NEXT_ACTIONS.md`; batch them through `/decision-record` when there are several.

## Phase 4 — Report + delivery

Validation report in `docs/` (dated header, byline, full blob URLs): encoding coverage of the source sections, per-rule accuracy table, contradiction triage, the `@DECIDE` inventory, and the interpretive-calls appendix (every `reading_note` surfaced in one place — this appendix is what makes the table citable in a paper). Branch + PR, merge per the handoff-scoped rule; model attribution tier + exact version. If the table should become citable data, hand off to `/data-release`.

## Hard rules

- No cell without a `source_ref`; no interpretation without its quoted wording + rejected alternative; no silence resolved without a human ruling.
- Accuracy numbers come from the actual validation run output, never estimated.
- The corpus validates the *encoding*, not the grammarian — a low-accuracy rule faithfully encoded stays in the table with its numbers, flagged, not "fixed" to match the corpus.

_Dr. Mārcis Gasūns_
