---
name: "export-consumer-smoke"
description: 'Real-consumer smoke test for an export profile — validator-green ≠ consumer-green; drive the flagship consumer (Lexique Pro, Systema hop…) and diff display/hop loss. Triggers: "smoke test the export", "does this render right in consumer".'
---

# export-consumer-smoke

Use this skill when the user asks to run the migrated source command `export-consumer-smoke`.

## Command Template


# /export-consumer-smoke <repo> <export-profile> [consumer-app]

_Created: 17-07-2026 · Last updated: 18-09-2026_

Census candidate C12 ([FABLE_FOOTPRINT_SKILLS_CENSUS.md](https://github.com/gasyoun/Uprava/blob/main/archive/FABLE_FOOTPRINT_SKILLS_CENSUS.md), refresh3, all-approve vote decided 17-07-2026; archived to `archive/` by H3101). Distills [H722](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H722-Fable_csl-standards_lexique-pro-mdf-smoke-test_11.07.26.md) (csl-standards, Lexique Pro 3.6/MDF) and [H727](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H727-Fable_SanskritLexicography_pwg-kosha-mdf-integration_11.07.26.md) (kosha, MDF-lift export family): a schema validator can pass an export file that still renders wrong, or not at all, in the app scholars actually open — [Uprava FINDINGS §69](https://github.com/gasyoun/Uprava/blob/main/FINDINGS.md) measured `validate-mdf-profile.mjs` passing 100% clean on an MDF file where Lexique Pro silently collapsed repeated `\bb`/`\lf`+`\le` markers, never rendered `\es` at all, and mojibake'd BOM-less UTF-8 under the wizard's default "Plain text" mode.

**Naming note:** the independent H1102 all-executor census (ruled in-chat, also 17-07-2026) proposed the identical shape under the name `/export-smoke-test` (item #26, same evidence — FINDINGS §69/H722). The review-sheet-voted name `/export-consumer-smoke` is authoritative since this handoff executed that formal vote; if `/export-smoke-test` ever gets separately built, treat it as a duplicate and redirect to this skill rather than maintaining two.

Argument: the repo owning the exporter, the export profile/format (MDF, TEI, CSV, whatever the repo emits), and the flagship consumer app if not already documented for that profile.

**Tier:** mechanical drive + judgment on what counts as meaningful display loss — Sonnet 5 (`claude-sonnet-5`)/Fable 5 (`claude-fable-5`) tier.

## Default path (seconds — run this first, byte/shape diff only)

```sh
python ~/Documents/GitHub/Uprava/tools/export_consumer_smoke.py \
    --export-cmd "<cmd that produces the export file>" --export-out <path> \
    --consumer-cmd "<cmd that reads it and produces a consumer-rendered file>" --consumer-out <path> \
    --baseline-report <path/to/prior/report.json> --apply
python ~/Documents/GitHub/Uprava/tools/export_consumer_smoke.py --selftest
```

LLM / judgment residue only after the script report exists (or the script exits non-zero with a real defect).

[`Uprava/tools/export_consumer_smoke.py`](https://github.com/gasyoun/Uprava/blob/main/tools/export_consumer_smoke.py) (H4102) runs the export command, then the consumer command, and computes byte/shape metrics (size, sha256, line count) for both output files, diffed against a prior JSON baseline report it itself wrote — verdict is `NEW-BASELINE` / `IDENTICAL` / `CHANGED`, report-only. Dry-run just prints the verdict; `--apply` also persists the current run as the new baseline for the next one. **Phase 1 (identify the real consumer), Phase 2 (get the consumer driveable), Phase 3's display-fidelity judgment (repeated-marker collapse, silent no-render, encoding auto-detect) and whether a `CHANGED` verdict is a regression or an intended schema change all stay LLM** — the script never drives a GUI and never grades a diff.

## Phase 1 — Identify the profile's real consumer

Every export format has one or two apps scholars actually open it in — not the validator. Check the profile's own docs first (e.g. `MDF_EXPORT_MAPPING.md`'s "Real-Consumer Smoke Test" section) for an already-identified consumer and a prior findings table before re-deriving from scratch; `/prior-art` against `PROJECT_INTERLINKS.md`/`SHARED_CODE.md` for an existing smoke harness on the same profile.

## Phase 2 — Get the consumer app driveable

- **Admin-gated installers:** `innoextract` unpacks a fully working portable layout without needing elevation (the H722 Lexique Pro pattern) — try this before asking for admin rights.
- **Owner-drawn / unreadable menus:** pywinauto's win32 backend can drive apps whose UI resists the standard accessibility tree — use keyboard jump-search plus coordinate clicks rather than fighting a menu tree that doesn't expose itself normally.
- Prefer the Browser pane / computer-use tooling already available in-session over standing up a new automation stack, unless the consumer is a native desktop app with no web equivalent (then pywinauto/computer-use per above).

## Phase 3 — Drive it, diff display-layer loss

Load the exported file into the real consumer and compare, field by field, against the source data — not against the export file's own schema (that's already validator-green by definition). Specifically hunt the failure classes §69 found, since they recur across formats:

- **Repeated-marker collapse** — a field that legitimately repeats (multiple `\bb`, stacked cross-refs) but the consumer only renders the first or last occurrence.
- **Silent no-render** — a marker/field type the consumer's schema accepts but never displays anywhere in the UI.
- **Encoding auto-detect failure** — BOM-less UTF-8 (or any non-default encoding) silently read as the wrong charset unless a wizard/import setting is set by hand.
- Any other consumer-specific quirk the profile's own prior findings table already names — check those are still reproduced, not just novel ones.

## Phase 4 — Report and document

A per-marker/per-field table: what the schema validator says (pass) vs. what the consumer actually shows (pass/degrades/silent-loss), with a screenshot or captured render where the consumer is graphical. Land this in the profile's own docs (the `MDF_EXPORT_MAPPING.md`-style "Real-Consumer Smoke Test" section, or create one if the profile has none) — a validator-only profile doc is incomplete once a consumer smoke test exists. Log any newly-found quirk to the owning repo's `FINDINGS.md` per `/findings-append`.

## Phase 5 — Fix or document, never silently patch the consumer

- If the loss is fixable on the export side (e.g. emit differently to avoid the consumer's collapse behavior), fix the exporter and add a regression case.
- If the loss is a genuine limitation of the third-party consumer app, document it as a known limitation in the profile docs — do not attempt to patch, reverse-engineer, or work around the consumer app itself; that's out of scope and out of the repo's control.

## Report

One line: repo, profile, consumer app, pass/degrade/fail count across fields tested, doc/FINDINGS links.

## Hard rules

- A schema validator passing is never sufficient evidence an export "works" — a new or materially-changed export profile isn't done until it has a consumer smoke test on record.
- Never modify the third-party consumer application; report its limitations, don't work around them destructively.

## Mode B — Pedagogy-export hop (H1660 U24)

Two-repo chain (do not invent a third skill):

1. **Producer** (usually SanskritGrammar): versioned adapter (`build_pedagogy_export.py` + `schema_version` + `--check` + tests) + LAST_MILE **Measured status** section — H1643.
2. **Consumer hop smoke** (usually Systema): load the public-safe feeds the hop expects; verify hop A/B/C (or the named hop) status is measured, not assumed; feature flag OFF until human `@DO` — H1644.

Rules:

- Validator-green on the producer is **not** hop-green.
- Pointer: reading-layer vocabulary/drill surfaces stay under `/reader-layer`; this skill owns **export fidelity into a live hop**.
- One hop letter or one producer schema bump per session unless the handoff names both halves of a paired pair.

## When this isn't the right tool

- Schema/structural conformance alone, no display-fidelity question — use the profile's own validator script.
- Designing a wholly new export format from scratch — that's the format-spec work itself; this skill only smoke-tests an exporter that already emits something.
- Building a full graded-reader layer (not just export hop) — `/reader-layer`.

## Scope — the carrier generalises beyond export profiles (U66, skill-mine H4542–H5153 vote 18-09-2026)

The discipline is "validator-green ≠ consumer-green"; the **carrier is any generated artifact, not only an export profile**:

- a **generated skill** — verify it through the actually-installed loader (opencode/codex), not the generator's own check (H5068: installed-consumer real-loader parity);
- a **mint outcome** — verify the uncertain outcome is preserved by every downstream consumer site (watchers, dashboards, close paths), never flattened to a success (H5067: C1/C2 watcher contract);
- a **registry row / manifest** — verify the real reader renders/consumes it, not just that the writer emitted it.

The procedure does not change: drive the REAL consumer over the artifact, diff what the consumer loses or silently rewrites (display collapse, dropped hop, flattened state), record the loss as evidence, and treat a green producer-side validator as no evidence at all. The original export-profile examples (Lexique Pro, Systema hop) stay as the worked precedents; this section only names the general carrier.

Evidence: H5068 · H5067. Related: [`/export-consumer-smoke`](https://github.com/gasyoun/claude-config/blob/main/commands/export-consumer-smoke.md) export profile mode above · [`/verify-claims`](https://github.com/gasyoun/claude-config/blob/main/commands/verify-claims.md) (claims, not consumers).

_Dr. Mārcis Gasūns_
