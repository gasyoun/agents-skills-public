---
name: "cologne-link-target"
description: 'Run the Dictionary-to-Book (DTB) link-target workflow for one Cologne dictionary — inventory ls citation abbreviations, map to scanned-edition pages via ls_resolver, validate sample links. Triggers: build scan links for DICT, wire ls click-throughs. Do not use for markup normalization or XML validation — that is /cologne-markup-batch or /cologne-xml-validate; this only wires ls-citation link targets to scanned-edition pages.'

---

# cologne-link-target

Use this skill when the user asks to run the migrated source command `cologne-link-target`.

## Command Template


# /cologne-link-target

_Created: 03-07-2026 · Last updated: 26-09-2026 (H5491: mechanical route ported)_

## Default path (seconds — run this first)

```sh
python3 ~/Documents/GitHub/Uprava/tools/cologne_link_target.py --dict PWG --orig-dir ~/Documents/GitHub/csl-orig --sample 20   # Phase 1 inventory: abbreviations, set-diff resolvable vs not, fix suggestions (READ-ONLY always)
python3 ~/Documents/GitHub/Uprava/tools/cologne_link_target.py --dict PWG --orig-dir ~/Documents/GitHub/csl-orig --out /tmp/ls_pwg_report.md
python3 ~/Documents/GitHub/Uprava/tools/cologne_link_target.py --selftest
```

LLM / judgment residue only after the script report exists — the inventory table and set-diff ARE Phase 1; the suggestions block pre-shapes Phase 2's worklist.

**The gate that stays: the link INTENT verdict.** The script can prove an abbreviation resolves (canonical `ls_resolver` prefix + pattern match) but cannot prove the mapped scan page shows the *cited passage* — that verdict needs the Phase 2 page-offset spot-checks (≥3 pages across the volume) and the Phase 3 frequency-weighted validation sample, and scholarly identification of unknown abbreviations routes to `/cologne-question-research`.

The one issue-taxonomy quadrant (milestone **Dictionary to Book**, labels `link-target` / `link-splitting`) without a dedicated runbook. The machinery already exists — **do not write a new resolver**.

Argument: dictionary code (e.g. PWG, MW, AP90), optionally an issue number.

## Canonical machinery (prior art — reuse, never rewrite)

- [`ls_resolver.py`](https://github.com/gasyoun/SanskritLexicography/blob/master/RussianTranslation/src/ls_resolver.py) in the SanskritLexicography repo (gasyoun, NOT sanskrit-lexicon — mind blob URLs; lives under `RussianTranslation/src/`) — the Python port of csl-app's `ls_service.dart`; already resolves `<ls>` source abbreviations to scan page links for the PWG article site.
- Upstream csl-app `ls_service.dart` and its per-dictionary source metadata — check what page-mapping tables upstream already carries for this dict before deriving one.

## Phase 1 — Inventory

From the dict's generated XML (or csl-orig source), extract all `<ls>` values; normalize and frequency-rank the abbreviation set. Separate: already-resolvable via ls_resolver / upstream metadata; combined refs needing splitting (`SOURCE N,N` → per-page, the `link-splitting` label); unknown abbreviations needing identification (that part is `/cologne-question-research` territory if scholarly).

## Phase 2 — Map

For unmapped sources: identify the scanned edition (archive.org / GRETIL / Cologne csldoc scans), then build the abbreviation → edition + page-offset table in the format ls_resolver consumes. Page-offset formula must be verified against at least 3 spot-checked pages spread across the volume (front, middle, back — pagination often shifts at gatherings).

## Phase 3 — Validate

Resolve a random sample (≥20, frequency-weighted) through ls_resolver and verify each lands on the cited passage. Record the sample and hit-rate — this goes in the deliverable.

## Phase 4 — Deliver

- Mapping table + any resolver-metadata additions: PR to the repo that owns them (SanskritLexicography for the local resolver; upstream csl-* repo if the mapping belongs there).
- If the fix requires touching csl-orig markup: route through `/cologne-correction-queue` — csl-orig takes no direct writes.
- Post ONE structured comment on the driving issue (validation stats, sample links, what remains) — minimal noise, edit-in-place on revisions.
- Label/milestone per the taxonomy; update `.ai_state.md`.

_Dr. Mārcis Gasūns_
