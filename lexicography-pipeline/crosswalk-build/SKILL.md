---
name: "crosswalk-build"
description: 'Build cross-source lexical crosswalks house-way: form_key() normalization, tiered matching, license check, kosha manifest; detector mode appends value+_basis columns to registers. Triggers: "crosswalk A to B", "verdict OCH-##".'

---

# crosswalk-build

Use this skill when the user asks to run the migrated source command `crosswalk-build`.

## Command Template


# /crosswalk-build <datasetA> <datasetB>

_Created: 07-07-2026 · Last updated: 26-09-2026 (H5494: mechanical route ported)_

## Default path (seconds — run this first)

```sh
python3 ~/Documents/GitHub/Uprava/tools/crosswalk_build.py --left a.tsv --right b.tsv --key-left headword --key-right lemma            # dry-run: per-tier counts (floor/relaxed/fuzzy), both denominators, key collisions — nothing written
python3 ~/Documents/GitHub/Uprava/tools/crosswalk_build.py --left a.tsv --right b.tsv --key-left headword --key-right lemma --out crosswalk.tsv --apply   # write crosswalk + per-side residue TSVs
python3 ~/Documents/GitHub/Uprava/tools/crosswalk_build.py --selftest
```

LLM / judgment residue only after the script report exists — the counts block IS Phase 2/3's numbers (tiered matching, both denominators, residue files); build only the gap on top of it.

**The gate that stays: the joined-side LICENSE `@DECIDE`.** The crosswalk inherits the MORE restrictive side's constraints (a CC BY-NC input makes the crosswalk BY-NC); genuinely ambiguous composition is never self-ruled — mirror the `@DECIDE` to `Uprava/GTD_NEXT_ACTIONS.md`. Fuzzy-tier promotion keeps its human gate too: the script emits `fuzzy-candidate` rows for review, never merged matches.

Distills the recurring crosswalk discipline from H021 (freq→kosha join), H048 (botanical), H082 (slp1 API mapping), H097 (GRA↔VedaWeb), H111 (Heritage forms), H128 (shared citation errors), H143 (PWG↔PWK Indische Sprüche). One session = one crosswalk TSV + coverage report, PR-delivered. Judgment tier (key collisions and joined-side licensing need real calls) — Sonnet 5 (`claude-sonnet-5`) / Opus 4.8 (`claude-opus-4-8`) / Fable 5 (`claude-fable-5`).

**Two modes.** *Build mode* (Phases 0–4 below) reconciles two datasets into a new crosswalk. *Detector-column mode* (its own section at the end) appends ONE feature detector's verdict as a column onto a crosswalk or claims register that **already exists** — a different discipline with a different failure surface. If the deliverable is a new column on an existing table, skip to that section; Phase 0 still binds.

## Phase 0 — Prior art (blocking)

Run the `/prior-art` escalation BEFORE any code. Canonical assets that already exist and must be **consumed, not rebuilt**: `union_headwords`, `mw_roots.tsv`, `corpus_lexicon`, the kosha manifest (`kosha/data/manifest/datasets.json`), and any existing crosswalk registered in `Uprava/PROJECT_INTERLINKS.md`. If a crosswalk between the same pair (or a superset) exists, extend it — a second parallel crosswalk over the same pair is a defect. State what exists, build only the gap.

## Phase 1 — Key normalization (the part that has burned us before)

- **Use the length-preserving `form_key()` from `sanskrit-util` / `scripts/sanskrit_util.py`** — lowercase, strip nom-sg visarga, fold anusvāra+ṅ/ñ/ṇ→n, drop vowel-pitch accents, PRESERVE vowel length and ś/ṣ.
- **NEVER the naive NFD → strip-Mn → NFC key**: it collapses `ā`→`a` (so `krānta`==`kranta`), kills retroflex dots, and collides ś with pitch-accented s — each of these fabricated false matches in prior sessions (see the IAST-pitfalls memory / `reference_iast_normalization_pitfalls`). The reader's `norm()`/`nfold()` are for diacritic-insensitive *search*, not for joining.
- State the join key explicitly in the report (SLP1 lemma, `form_key(IAST)`, dict-siglum × headword, …) and apply the **identical** normalizer to BOTH sides — asymmetric normalization is the classic silent-mismatch source (BOM/whitespace handling included).

## Phase 2 — Tiered matching

Match in declared tiers, each with its own count — never one opaque fuzzy pass:

1. **Floor tier** — exact match on the normalized key. This is the trust anchor.
2. **Relaxed tier(s)** — declared, individually counted relaxations (e.g. final-vowel variance, avagraha, hyphenated compounds split). Each relaxation is named in the report with its match count.
3. **Fuzzy tier** — edit-distance or heuristic candidates, output as *candidates for review*, never silently merged into the floor. Route a human gate through `/review-sheet` when the fuzzy tier will feed anything downstream.

Output: a crosswalk TSV with a `tier` column, one row per mapping, plus an unmatched-residue file per side.

## Phase 3 — Coverage report (honesty is the deliverable)

A `docs/`-level report (dated header, byline, full blob URLs) with: per-tier counts and percentages against BOTH sides' totals, the key-reconciliation caveats stated plainly (what the normalizer folds away, what collides, known homograph classes), sample spot-checks per tier, and the residue characterized (why the unmatched are unmatched — coverage gap vs genuine absence). Per the log-tables convention, the counts table lives in this committed report, not only in chat. A coverage number without its tier breakdown is a defect.

## Phase 4 — License + registration

- **Joined-side license check**: the crosswalk inherits the MORE restrictive side's constraints (a CC BY-NC input makes the crosswalk BY-NC). Genuinely ambiguous composition → `@DECIDE`, never self-ruled. If the crosswalk should become a citable release, hand off to `/data-release`.
- Register the crosswalk as a feed in `Uprava/PROJECT_INTERLINKS.md` and add/update its row in `kosha/data/manifest/datasets.json` in the same pass.

## Delivery

Branch + PR in the owning repo, merge per the handoff-scoped rule. Model attribution tier + exact version in the report's provenance line. Mirror any license `@DECIDE` to `Uprava/GTD_NEXT_ACTIONS.md`.

## Hard rules

- No NFD+strip keys, ever. No undeclared relaxations. No fuzzy matches promoted without a human gate.
- Consume canonical assets; never re-derive `union_headwords`-class data locally.
- The report states what the crosswalk CANNOT distinguish, not just what it matched.

---

# Detector-column mode

_Added 17-07-2026 from H855 (kosha dhātu resolver), H978 (ряд-1978 columns), H1000 (period map), H1001 (causative grade), H1004 (compound type), H1008 + H1012 (root shape), H1022 (treebank syntax)._

**One detector per pass.** The deliverable is a feature detector plus the column its verdicts occupy on an **existing** crosswalk/register — never a rebuild of the table, never two detectors in one pass. Model tier: **Opus 4.8 (`claude-opus-4-8`)** — the probe ruling, the confound controls and the honest-UNTESTABLE calls are the whole skill; a mechanical re-run of an existing detector is Sonnet 5 (`claude-sonnet-5`) tier.

## Phase D0 — The blocker names the instrument

The pass starts from an existing row whose verdict is `UNTESTABLE` **and whose own text names the missing instrument** — not from "what could I measure". A register whose UNTESTABLE block holds vague claims rather than named instrument gaps is not ready for this mode. Each pass's residuals must name the next pass's detector (H1000 surfaced the causative detector; H1001 built exactly it).

## Phase D1 — Probe the data layer BEFORE writing the detector

The most consistently load-bearing step. Establish what the source data can actually support, then record the ruling as a dated block in the script's docstring (`DETECTION DESIGN (probed <date> before building)`). The probe routinely inverts the design:

- H1001 found DCS lemmatizes causatives as their own `-ay` lemmas → detection is **lemma-level, no form matching**.
- H1004 found DCS tags compound members lexically → compound **function is unrecoverable**, so the tagger is structural only.
- H1022 found the premise itself was false — the four claims' blocker ("needs syntactic annotation the export lacks") did not survive a token-level look. **A blocker that was never probed is a presumption, not a blocker**; probe before you inherit one.

## Phase D2 — Operationalize in the source's own terms

Encode the source's own rules and named lists; **never re-derive them independently** (H978: the §66/§67 rules and named-root lists override the 2014 series where they speak — no independent phonological re-derivation). Where a section names its own exceptions, each exception gets a slot — H1008's register had **explicitly refused a naive regex**. Honor the source's own parentheticals: H1004 excluded eka-first from dvigu and **counted it apart** (1,875) rather than dropping it silently.

## Phase D3 — Self-test on the source's own worked examples, before the full run

Non-negotiable, and it pays every time: H1012's 15-case self-test replicated every worked example in §59 and **caught two bugs before the run**; H978 validated 15/15 against §§66–68's example roots. Self-test failures accumulate into the stats sidecar (`self_test_fails`), never into a silent pass.

## Phase D4 — Run the full frame and declare BOTH denominators

Always the raw rate **and** the resolved rate, each named: `fit_rate_pct_of_all` 97.6% alongside `fit_rate_pct_of_resolved` 98.4% ("of the 739 roots where a nucleus could even be located"). State corpus coverage as a number (H1000: 4.07M tokens = 71.6%), and pre-register the floor in the goal line when one exists.

## Phase D5 — Confound controls: exclusion-with-reason + an internal control

- Every exclusion carries a per-item reason in the stats JSON — `"Rājanighaṇṭu": "nighaṇṭu (lexicon): word lists, not sentences"` — never a bare filter.
- Build an **internal control column** that would fire if the detector were measuring the wrong thing. H1000's epic-imitative purāṇa control landed between epic and classical (48.1%), showing the map measures language rather than genre labels; H1001 independently re-hit it at 26.3%.
- Try to rescue the claim before verdicting against it, and report the failed rescue (H1022's genre-skew check: "the skew does **NOT** rescue the claims — the classical bucket is if anything WEAKER").

## Phase D6 — Verdict the row, inventory the residue

Flip the verdict, rewrite the `number:` field with the measurement (opening `MEASURED with … built <date> (<script>.py, H####)`), point `ref:` at the stats JSON + script, regenerate the register from its source of truth (`npm run claims` — `CLAIMS_VERIFIED.md` is GENERATED, never hand-edited), re-tally. Residue is **individually documented, never a silent cap** (H1012's 18 non-fits: 9 genuine unusual clusters, 6 catalog/tradition citation mismatches, 3 extra-syllable roots).

## The column contract

- **Name the column `<feature>_<sourceyear>`.** The year suffix is what stops two taxonomies' same-named features from colliding — `jan` is N₂ per §66 but N₁ in the 2026 column. **Never equate two taxonomies' same-named columns.**
- **Every verdict column carries a `_basis` sibling** naming the rule that fired: `§66-rule` · `§67-A1-list` · `§67-A1-default`. Named-list beats rule-default; the basis records which.
- **Closed vocabularies only** — `openness_1978 ∈ {open, closed}`, `polnoizm_1978 ∈ {full, defective}`. Method/provenance fields likewise: kosha's `via ∈ {3sg, direct, bare, unresolved}`.
- **`unresolved` is a first-class value, not a null**, and it is tallied (`via_counts: {3sg: 474, direct: 205, bare: 43, unresolved: 57}`, `resolved_pct: 92.7`). Declare a **fallback ladder ending in `unresolved` — never guess a resolution** (H855).
- **Four distinct uncertainty states, never collapsed:** `PENDING` (harvested, unverified) · `UNTESTABLE` + named missing instrument · `UNTESTABLE` + **measured probe** ("no instrument, now PROBED rather than presumed") · `UNTESTABLE` + **negative pilot** (the dead end recorded with its numbers so nobody re-derives it).
- **One `<row-id>_*_stats.json` sidecar per detector**, opening with an `instrument` sentence (script + input + snapshot pin + the design caveat), then metrics, then a `validation`/`self_test` block of named boolean checks, plus `excluded` with reasons.

## Hard rules (detector mode)

- **Annotation density is not language.** DCS verbal-feature annotation collapses for later texts (tagged ta-participles RV 1,874 → Daśakumāracarita 0, measured 16-07-2026 in H1000), so feats-based metrics measure annotation density. Restrict verdicts to annotation-robust layers — lemma / upos / mwt / text identity — and mark any metric resting on sparse `feats` as conditional at the metric level.
- **Structural ≠ functional.** Say which one you built, in the detector's name and its docstring, and leave the functional claim honestly UNTESTABLE with a measured blocker (H1004).
- **Composition bias disqualifies an aggregate proxy.** Record it as a negative pilot, do not trust it (OCH-24: "recorded, not trusted").
- **Anchor the detector on an independent existence check** to screen the confound class — H1001 required the root lemma to exist as a VERB lemma (`janay → jan` exists; `kathay` excluded), screening denominatives; the residual risk is documented, not hidden.
- **One canonical instrument, imported.** H1001 sliced by H1000's period map imported from `period_style_gradient.py` — one canonical map, not two copies.
- **The detector column improves the COMPARISON only; it never becomes the display base** (H855 guardrail D3).
- **In-place column maintenance is idempotent and touches nothing else** — re-running recomputes only its own columns; duplicates preserved; the filename stays put for citation stability even when its contents grow (H978 kept `morphoclass_crosswalk_1975_2014_2026.csv` after adding 1978 columns).
- **Pin the corpus snapshot** in every instrument sentence (`dcs_full.sqlite`, dcs-conllu `04e0778`). Bulk source data is build-time-only and gitignored (H855 R12); the committed artifact carries strings.
- **Fail loud on a map/DB mismatch** — `assert not missing, f"period-map names absent from DB: {missing}"`, never a silent empty slice.
- **Errata found while encoding are recorded, not silently fixed** (H978's two print-corruption suspects live in the script header with the encoding decision stated).
- **Flag in advance, then measure.** Naming the vulnerable claim before any number exists is what makes the result a test rather than a rationalization (OCH-68, which duly came back OVERSTATED).

## Delivery (detector mode)

One PR → merged → a version bump + release tag per detector → CHANGELOG + README follow-through; quote the register's running tally as the pass's receipt (`74 verified = 63 TRUE · 0 OVERSTATED · 0 FALSE · 11 UNTESTABLE`). Endgame condition: every remaining UNTESTABLE carries a **measured** blocker or an external-data dependency — no presumed blockers left.

_Dr. Mārcis Gasūns_
