---
name: "hypothesis-probe"
description: 'Run ONE numbered corpus hypothesis (HB-###/HK-### row in claims.yml) end-to-end — probe script + JSON sidecar, verdict to register, outcome to QUESTIONS_LOG. Triggers: "run HB-###", "verify this claims.yml row". Mechanical — Sonnet 5 (claude-sonnet-5).'

---

# hypothesis-probe

Use this skill when the user asks to run the migrated source command `hypothesis-probe`.

## Command Template


# /hypothesis-probe [book-dir] [hyp-id]

_Created: 17-07-2026 · Last updated: 28-08-2026_

**Script-first:** the mechanical phases run through
`python Uprava/tools/hypothesis_probe_run.py` (census `list`, sidecar contract
`validate-sidecar`, Phase 4 `write-back`) — LLM / judgment residue only after
the script report exists: the prior-art walk (Phase 1), the probe-instrument
authoring (Phase 2), and the verdict ruling itself (Phase 4 — the tool's
`--verdict` is a required argument the calling session rules first; it never
invents one).

The SanskritGrammar per-book hypothesis-probe discipline, generalized from eight sessions
that each ran ONE numbered claim to a corpus verdict by hand (H1035, H1040, H1042, H1044,
H1045, H1047, H1056, H1060). Where `/grammar-claims-audit` owns the register LIFECYCLE
(harvesting and tracking many claims) and `/census-stat` computes descriptive statistics,
this skill owns the RUN of a single already-registered hypothesis, end to end.

Argument: an optional book directory (`BuhlerLeitfaden_1923/`, `KocherginaUchebnik_1998/`,
…) and/or a specific `HB-###`/`HK-###` ID; if omitted, pick one `UNTESTABLE` row from the
book's `claims.yml` (or an explicit re-verdict target the user names).

Evidence base: H1035 H1040 H1042 H1044 H1045 H1047 H1056 H1060 (8 sessions, SanskritGrammar).
Dedupe: `/grammar-claims-audit` (register lifecycle, many claims) · `/census-stat`
(descriptive stats, no claims.yml write-back) · `/hypothesis-mint` (mints/resolves
QUESTIONS_LOG rows, never runs a probe itself) — `/hypothesis-probe` is the one that
actually executes a numbered claim.

## Phase 0 — Pick one hypothesis

Script report first: `python Uprava/tools/hypothesis_probe_run.py list --book <dir>`
(tally + per-row list; `--verdict UNTESTABLE` filters, `--id HB-###` prints one
block). Source the per-book `claims.yml` register and pick ONE row whose `verdict_fact` is
`UNTESTABLE` (or an explicit re-verdict target). **One session = one hypothesis** — never
batch several rows in a single pass; that is what made the 8 evidence sessions tractable
and reviewable one PR at a time. Which row is worth probing stays a judgment call —
the tool tallies, it does not choose.

## Phase 1 — Prior art before any fresh corpus query

Check whether an existing data asset or a prior probe's identification technique already
covers this before writing a new query — reuse is method/asset-level, copy-adapted, never
a shared helper library (there is no such library in this genre by design). Every one of
the 8 exemplars reused something: HK-5 reused HK-207's aorist typing, HK-42 reused the
animacy sembank lookup, HK-88 reused the H978 crosswalk CSV — zero of them ran a fresh
query from scratch.

Known data sources: primary `../VisualDCS/src/DCS-data-2026/dcs_full.sqlite`
(223,751-token parsed slice, UD-style feat columns); non-DCS assets:
`TolchelnikovTalmud_2026/data/morphoclass_crosswalk_1975_2014_2026.csv`,
`../dcs-conllu/lookup/` (WordNet sembank).

## Phase 2 — Write the probe instrument

One standalone stdlib Python script, landing in the book directory, **named for the
phenomenon it measures, not the hypothesis ID** (`gender_resolution_probe.py`,
`two_grade_root_vowel_check.py` — never `hk42_probe.py`):

- The docstring IS the report: claim restated verbatim, method, `RESULT (DD-MM-YYYY): …`,
  Usage.
- `sys.stdout/stderr.reconfigure(encoding='utf-8')`, no UTF-8 BOM, `HERE =
  Path(__file__).resolve().parent`, a `DEFAULT_DB` constant + argparse `--db` override.
- **Produce-not-decide**: the script emits counts/tables only — the verdict is ruled in
  Phase 4, never hard-coded inside the script.
- **Anchored patterns only.** Unanchored regex stems are the canonical trap of this exact
  probe genre — see the Renou classifier defect (`project_renou_classifier_defect`
  memory: unanchored regex stems, ≥7.1% wrong).
- State the denominator explicitly and check genre skew before verdicting — a density
  ratio can be a denominator artifact (this is what produced the HK-5 OVERSTATED call).

## Phase 3 — JSON sidecar

Emit `<hypid>_<slug>.json` next to the script. First key `"instrument"` — a one-line
self-description INCLUDING the caveat. Last key a `confirmed`/`conclusion` boolean+prose.
`json.dump(..., ensure_ascii=False, indent=2)`. Then prove the contract mechanically:
`python Uprava/tools/hypothesis_probe_run.py validate-sidecar <hypid>_<slug>.json`
(checks first/last key, no BOM, ensure_ascii=False, indent=2 byte round-trip) — the
verdict content inside stays the session's to judge.

## Phase 4 — Verdict + write-back

Rule the verdict FIRST (TRUE / FALSE / OVERSTATED / stays UNTESTABLE / SIZED — thin-n
honesty is the discipline's core value: never manufacture a verdict to close a row),
then hand the mechanics to the tool:

```
python Uprava/tools/hypothesis_probe_run.py write-back --book <dir> --hyp-id HB-### \
    --verdict TRUE --script <probe>.py --summary "..." [--handoff H####] [--apply] [--regen]
```

Dry-run by default (unified diff); `--apply` performs the surgical `claims.yml` edit
(`verdict_fact` flip + `number:` rewritten as dated provenance `MEASURED (DD-MM-YYYY,
<script>.py, H####): …`; `SIZED` keeps `UNTESTABLE` with the
`SIZED, not "no instrument" (…)` prefix and is idempotent — a repeat is a loud NO-OP);
`--regen` best-effort runs the book repo's own `scripts/build_claims.py` +
`claims_schema_validate.py` (loud skip when its PyYAML dep is missing).
`CLAIMS_VERIFIED.md` + `claims.json` are generated, never hand-edited.
Known book-side drift (28-08-2026 census): the Buhler claims.yml header synthesis tally
(«381 TRUE · 7 UNTESTABLE») is stale prose — live register is 383/5; trust the tool's
tally, fix the header in the book repo separately.

## Phase 5 — Deliver

- CHANGELOG bullet (a fresh measured verdict → `### Added`; a sizing note or a flip →
  `### Changed`) + book patch-version bump; Docusaurus build-verify.
- Branch `h<id>-<slug>`, squash-merged PR, commit subject idiom `H####: <probe name> —
  <HYP-ID> <verdict>` (all 8 exemplars: PRs #272 #281 #285 #290 #294 #297 #304 #309).
- **Wire the verdict to [Uprava/QUESTIONS_LOG.md](https://github.com/gasyoun/Uprava/blob/main/QUESTIONS_LOG.md)**
  as a `[T]` row via `/hypothesis-mint` resolve-mode. All 8 evidence exemplars skipped
  this step (zero probe IDs appear in QUESTIONS_LOG) — this skill closes that gap; the
  wiring is part of the discipline on every run, not an afterthought.

## Gotchas (from the 8-session evidence base)

- Thin-n honesty: `SIZED, not "no instrument"`, never a manufactured verdict (HB-158 n=4,
  HK-207 no drama texts in DCS, HK-42, HK-86).
- Candidate-identification contamination: lexicalized false-privatives pattern-matching
  `a-/an-+lemma` inflate false positives if not filtered.
- Wrong-denominator-node: narrow WordNet `abstraction` (30.6%) vs the full Tops set
  (52.8%) give different answers to the same question — state which one you used.
- Ontology holes: e.g. *deva* classed under `causal_agent` not `person` — spot-check the
  classifier's buckets before trusting them.
- Comparison corpus absent → fall back to a genre proxy with a loud caveat, checked
  against the full 270-text DCS inventory.
- Cross-export figure mismatch: DCS-2021 CSV vs 2026 sqlite tense codes differ — never
  claim a reproduction across exports.
- `feat_tense='Past'` collapses aorist/perfect/pluperfect; only 7 `feat_formation`
  subtypes are aorist-specific.
- Kochergina is in-copyright: aggregate numbers only in registers, registers stay `.md`,
  no site page.

Model provenance on every verdict + CHANGELOG bullet: tier AND exact version.

_Dr. Mārcis Gasūns_
