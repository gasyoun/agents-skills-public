---
name: "statistical-error-class-redteam"
description: 'Red-team derived numbers against the fixed statistical-footgun taxonomy (denominator artifact, confounds, fallback buckets) to per-claim verdicts, defects re-verified. Fable 5 (claude-fable-5), Opus 4.8 (claude-opus-4-8) fallback. Trigger: "statistical audit".'

---

# statistical-error-class-redteam

Use this skill when the user asks to run the migrated source command `statistical-error-class-redteam`.

## Command Template


# /statistical-error-class-redteam <paper / dataset / report>

_Created: 20-07-2026 · Last updated: 02-09-2026_

Every one of the error classes below was discovered **independently, the hard way, in a
separate session** — and nothing encoded the checklist, so the next paper rediscovered the
next class from scratch. The «9× density» figure that turned out to be a denominator
artifact ([H800](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H800-Fable_SanskritGrammar_gradation-metalanguage-register-and-zaliznyak-3work-comparison_12.07.26.md));
the Titov parametric-core effect that was really article size
([H457](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H457-Opus_VisualDCS_titov_parametric_core_dictionary_side_10.07.26.md));
the Renou classifier defect that made every per-register count uncitable
([H459](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H459-Sonnet_IndologyScholars_renou-rules-anchor-fix-and-dedupe_10.07.26.md));
the fallback-bucket trap ([FINDINGS §125](https://github.com/gasyoun/Uprava/blob/main/FINDINGS.md)).
This skill is that accumulated scar tissue as a **reusable lens set**, distilled from
[H1269](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1269-Fable_Uprava_portfolio-statistical-redteam-error-classes_18.07.26.md)
Fable 5 (the six-class checklist + CLEAN/SUSPECT/CONFIRMED-DEFECT verdicts),
[H1229](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1229-Fable_SanskritGrammar_dcs-derived-numbers-adversarial-rederivation_18.07.26.md)
Fable 5 (the re-derivation ledger) and
[H1272](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1272-Fable_SanskritGrammar_gold-provenance-contamination-audit_18.07.26.md)
Fable 5 (gold provenance).

Argument: the paper (`Axx`), dataset, or report whose derived numbers are under audit.

## Phase 1 — Inventory the derived numbers FIRST

Extract every number a claim rests on — percentage, count, ratio, κ, coverage figure,
"N× more" — into a ledger **before** running any lens, with **where it is computed** (the
script/notebook/query, not just the sentence citing it). Building the checklist after
reading the analysis inherits the original author's blind spots. A number whose computation
site cannot be located is itself a finding: mark it `UNTRACEABLE` and say so.

## Phase 2 — Run the fixed taxonomy, every lens against every number

| # | Error class | What it looks like | The probe |
|---|---|---|---|
| 1 | **Denominator artifact** | "N× denser / more frequent" where the effect lives in the denominator, not the phenomenon | Restate the denominator explicitly; recompute both sides on a common base |
| 2 | **Size/length confound** | The effect tracks article/text length rather than the parameter claimed | Stratify by size band, or regress length out; does the effect survive? |
| 3 | **Classifier-defect contamination** | Per-class counts inherit a known classifier error rate | Look up the classifier's *measured* error rate; does the claim survive it? |
| 4 | **Fallback-bucket trap** | Items landing in a default / "other" / unknown bucket get read as a positive finding | What fraction is fallback, and does the claim depend on that fraction? |
| 5 | **Coverage overstatement** | "covered / aligned N%" where the denominator quietly excludes the hard cases, or a silent top-N cap | Name what was dropped and whether the drop was logged |
| 6 | **Single-run fragility** | One seed / run / model pass reported as a stable number | Re-run, or state the variance — a single run is not a measurement |
| 7 | **Gold-provenance contamination** | A "gold" set that is actually model-vs-model, so every κ measures agreement between two models | Trace the annotation provenance to a human; if none, the κ is not inter-annotator agreement |

Lens 7 is the newest and the most load-bearing org-wide: the provenance audit found **0
genuinely-gold sets among 15 org datasets labelled "gold"**. Never accept a κ without
tracing who (or what) produced both sides.

## Phase 3 — Per-claim verdict, independently re-verified

Every number gets exactly one verdict: **CLEAN** (survived all seven lenses) · **SUSPECT**
(a lens fires but the defect is not yet reproduced) · **CONFIRMED-DEFECT** (reproduced).
Promote SUSPECT → CONFIRMED-DEFECT **only by reproducing the defect** — recompute the
number the correct way and show both figures. Use the `adversarial-verifier` agent per
candidate defect, and where a number can fail through more than one lens, verify through
distinct lenses rather than repeating one. Record the corrected figure next to the original;
a bare "wrong" without the right number is not actionable.

## Phase 4 — Route the confirmed defects, and persist the table

- The verdict table is a substantive result → persist it in the same pass (the owning paper's
  doc or `RESULTS_LOG.md`; cross-repo measurements via `/findings-append`), with the date,
  one-line context, and the auditing model as tier + exact version.
- A **new** error class discovered during the sweep goes to
  [FINDINGS](https://github.com/gasyoun/Uprava/blob/main/FINDINGS.md) **and** gets added to
  the table in Phase 2 — that is how this taxonomy grows instead of being rediscovered.
- Each CONFIRMED-DEFECT gets its own fix: a correction to the paper, an errata entry, or a
  GTD `@DO`. If the defective number is already published or cited downstream, flag the
  consumers in the same pass.

## When this isn't the right tool

- Re-executing ONE agent session's whole work-pass → `/adversarial-pass-review`.
- ONE numbered grammar/lexicon claim vs a corpus proxy → `/claim-verify`.
- Refereeing the whole paper (argument, structure, venue fit) → `/paper-referee`.
- Producing a census/statistic in the first place → `/census-stat`.

## Compatibility Rule

The canonical source of this skill is `~/.claude/commands/statistical-error-class-redteam.md`
(the claude-config repo); Codex and Grok twins are GENERATED from it by
`scripts/generate_playbook_twins.py statistical-error-class-redteam` and must never be hand-edited.

_Dr. Mārcis Gasūns_
