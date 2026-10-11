---
name: "gold-adjudicate"
description: 'Adjudicate two independent gold annotation passes — verify genuine passes, compute agreement, rule disagreements from source records, emit adjudicated gold + precision/recall. Opus 4.8 (claude-opus-4-8)/Fable 5 (claude-fable-5). Trigger: "compute the kappa".'

---

# gold-adjudicate

Use this skill when the user asks to run the migrated source command `gold-adjudicate`.

## Command Template


# /gold-adjudicate

_Created: 03-07-2026 · Last updated: 01-10-2026_

The MWS-G5 / pwg_ru-judge pattern, generalized: two independent passes exist so their DISAGREEMENTS locate the hard cases — adjudication is reading those cases in the source, not vote-counting. With MW-as-template, every dictionary will eventually run this.

Argument: pointers to the two pass artifacts (files, PRs, or the handoff that names them); resolve from GTD/handoffs if omitted.

## Phase 0 — Validate the inputs (do not skip)

Before any statistics, confirm for each pass:

- **It is a genuine per-record reading pass, not a rule-based script's output.** This trap already happened live (H070, Fable 5 — MWS G5 adjudication: Pass B started as a script and was corrected mid-session). Check the pass's provenance memo/PR description AND spot-read ~5 annotations against their records — rule-derivable labels with no evidence of reading are the tell. A rule-based arm invalidates the design; stop and report rather than adjudicate garbage.
- **Same sample frame** — identical record IDs, same N, same sampling seed if one was pinned. Any frame mismatch → stop.
- **Same label schema** — map trivially-renamed labels; a semantic schema drift is a stop.
- **Model provenance** — tier + exact version of each pass's annotator, recorded now for the memo.

## Phase 1 — Agreement

Write a script to a `.py` file (never inline; `sys.stdout.reconfigure(encoding='utf-8')`; `encoding='utf-8'` everywhere, no BOM) that computes: raw agreement, per-label confusion matrix, Cohen's κ. Commit the script next to the data — the numbers must be recomputable.

## Phase 2 — Adjudicate the disagreements

For EVERY disagreement, read the underlying source record and rule: A correct / B correct / both wrong (adjudicated third label) / genuinely ambiguous (record as such — an honest ambiguous bucket beats a forced call). One line of reasoning per ruling, kept — the reasoning is what feeds the paper's error analysis. Tag each ruling with a per-item outcome dot for the memo table (this grades *how clean this ruling was*, not a finding's importance): 🔵 clear call, one pass simply wrong · 🟡 ruled but genuinely close, or the "both wrong → adjudicated third label" case · 🔴 recorded as genuinely ambiguous — no ruling made, flagged for the error-analysis section rather than folded into gold silently. Also spot-check a sample of the AGREEMENTS (~10%): two passes agreeing on a wrong label is the failure mode agreement stats can't see.

## Phase 3 — Outputs

- **Adjudicated gold file** — same schema as the passes, plus `ruling` + `ruling_reason` columns, committed where the passes live.
- **Per-pass P/R** against the adjudicated gold, per label and overall.
- **Memo** — `<task>_adjudication_<DD-MM-YYYY>.md` next to the gold: dated header + byline, full blob URLs, κ + confusion + P/R tables, the ambiguous-bucket list, and a provenance block naming tier + exact version for Pass A, Pass B, and the adjudicator (three different models is the healthy case — say which was which).

## Phase 4 — Deliver

Branch + PR + auto-merge. Fold the measured numbers into the consuming paper if one is named (that's usually the whole point — e.g. A16 waiting on G5 P/R): update the manuscript's placeholder figures, or open a GTD row if the fold-in is its own session. Close the handoff via [`/handoff-close --reconcile`](https://github.com/gasyoun/claude-config/blob/main/commands/handoff-close.md) (H1342, Opus 4.8 — bookkeeping batching/WIP cap: the row flip + close clause buffer into the session's single `registry_reconcile.py flush`) and update GTD.

## Pool mode — N>2 annotators (U68, 01-10-2026)

The dual-pass core above generalizes to a pool. Distilled from epic E014 (H5307 design → H5308 packets→double-keyed sheets + pool validator → H5309 κ-report tool → H5312 OBS-T blind second-annotator 390 rows + fail-closed `--ingest` → H5313 κ folded into A31 → H5330 300×7 stratified double-keyed sheets → H5483 merge B7 + Krippendorff/Landis-Koch B8); approved **U68** (MG chat vote 10/10, 01-10-2026):

- **Blind by construction** — seed-pinned assignment manifest with rotating offsets; each annotator sees only their own slice, annotators never see each other's labels, and a `do-not-annotate` dry-run declaration precedes any real pass.
- **Multi-rater statistics beside pairwise** — Cohen's κ per dyad (≥30 rows per dyad or say so) + Fleiss κ / Krippendorff's α headline + Landis-Koch bands; raw agreement and the confusion table stay in the report.
- **Merge truth-table** — five named cases, ties NEVER default to the machine, contact-token refusal for identity-sensitive rows; every merge decision is ruled, not averaged.
- **Honesty rules** — n=10 R2 cannot support a reliability claim; no pooled α across heterogeneous tasks; low κ is reported as measured, never softened ("low κ reported as measured" — H5313); an expert baseline is kept separate, not merged into the pool.

_Dr. Mārcis Gasūns_
