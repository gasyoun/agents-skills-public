---
name: "sense-corpus-join"
description: 'Join dictionary sense inventories to corpus attestation/frequency - length-preserving keys, both denominators, join TSV + coverage. Triggers: "sense frequency", "corpus attestation per sense". Sonnet 5 (claude-sonnet-5); Opus 4.8 (claude-opus-4-8) judgment.'

---

# sense-corpus-join

Use this skill when the user asks to run the migrated source command `sense-corpus-join`.

## Command Template


# /sense-corpus-join <dict-side> <corpus-side> [axis]

_Created: 23-07-2026 · Last updated: 26-09-2026 (H5495: mechanical route ported)_

## Default path (seconds — run this first)

```sh
python3 ~/Documents/GitHub/Uprava/tools/sense_corpus_join.py --dict senses.tsv --corpus freq.tsv --key-dict headword --key-corpus lemma            # dry-run: both denominators, residues, conflict list — nothing written
python3 ~/Documents/GitHub/Uprava/tools/sense_corpus_join.py --dict senses.tsv --corpus freq.tsv --key-dict headword --key-corpus lemma --out join.tsv --apply   # write join TSV (matched + dict_only + corpus_only) + conflicts TSV
python3 ~/Documents/GitHub/Uprava/tools/sense_corpus_join.py --selftest
```

LLM / judgment residue only after the script report exists — the coverage block IS Phase 3's numbers (both denominators, first-class residues); the join TSV is Phase 5's deliverable core.

**The gate that stays: conflict ADJUDICATION.** The script reports duplicate normalized keys (spelling/homograph collisions, e.g. `saṃbhavaḥ`/`sambhavaḥ` folding together) and stem-vs-lemma asymmetries — it never resolves them by guessing; each collision is a judgment call (true homograph? spelling variant? different units?) that a human or judgment tier owns. Corpus snapshot pinning (Phase 1) and lacunar lower bounds (Phase 4) stay with the skill too.

Census shortlist H1531 (post-H1097 footprint, no vote gate — MG direct). Encodes the recurring **dictionary sense/lemma × corpus** join that `/crosswalk-build` does not own: crosswalks reconcile two *dictionaries* (or a detector column on an existing table); this skill joins a **dictionary inventory** to a **corpus attestation/frequency feed** and reports the divergence honestly.

Evidence base: H1328 (MW uttarapada × DCS Kompozity dict-vs-corpus), H1389 (union corroboration / text-attestation regrade), H1397–H1398 (uttarapada divergence + trainer ranking), H1453 (wordsem 3-layer sense frequency), H1455 (per-sense corpus attestation join), H1459 (genre stratification wave 2), H1483 (definition typology), H1484 (coverage ribbon / lemma–entry timeline), H1587 (sense-recon pilot), H1588 (two-witness WSD), H1212 (primary-source attestation). Modes extended H1617 vote U21.

Argument: the dictionary-side inventory (headword list, sense table, uttarapada index, definition loci export, …), the corpus-side feed (DCS token/lemma tables, wordsem layers, genre-tagged attestation, …), and optionally the analysis axis (`dict-vs-corpus` · `per-sense-freq` · `genre-strat` · `timeline` · `definition-typology` · `sense-recon` · `wsd-two-witness`).

## Phase 0 — Prior art (blocking)

Run `/prior-art` before any join code. Canonical assets that already exist and must be **consumed, not rebuilt**:

- kosha manifest (`kosha/data/manifest/datasets.json`) and any registered lemma/sense crosswalks
- VisualDCS / dcs-conllu derived masters (pin + SHA-256 — FINDINGS §87: a commit pin alone is not durable if history rewrites)
- `union_headwords`, sense-export tables, uttarapada indexes already committed
- rows in `Uprava/PROJECT_INTERLINKS.md` naming this pair

If a join between the same pair (or a superset) already exists, **extend** it — a second parallel join over the same pair is a defect. State what exists; build only the gap.

## Phase 1 — Freeze both sides

1. **Dictionary side.** Identify the unit of join: headword, sense ID, sense locus (definition span), compound member (uttarapada/pūrvapada), or definition typology class. Export a stable TSV with an explicit key column and a stable sense identifier where senses exist.
2. **Corpus side.** Identify the attestation unit: token, lemma, sense-tagged occurrence, genre bucket. Prefer a committed derived table over re-streaming the full treebank.
3. **Pins.** Record corpus snapshot date, source commit or master SHA-256, and the dictionary export as-of. Refuse to proceed without a pin (the H1083 / §87 contract).

## Phase 2 — Key normalization (same trap as `/crosswalk-build`)

- Use the length-preserving `form_key()` from `sanskrit-util` — never NFD→strip-Mn→NFC.
- Apply the **identical** normalizer to both sides.
- State the join key in the report (SLP1 lemma, sense-id, form_key(IAST), …).
- When the corpus tags stems but the dictionary lists full lemmas (or vice versa), declare the asymmetry and count the residue — do not silently collapse it.

## Phase 3 — Join + both denominators

Always report **both** denominators:

| Count | Meaning |
|---|---|
| `|dict ∩ corpus| / |dict|` | share of dictionary units attested in the corpus |
| `|dict ∩ corpus| / |corpus-units-in-scope|` | share of corpus units that land in the dictionary inventory |

Never publish only the flattering side. Unmatched residues on each side are first-class output columns (`dict_only`, `corpus_only`), not dropped rows.

Axis-specific extras:

- **`dict-vs-corpus`** — Jaccard / zero-overlap rates per stratum (H1328: same finals, near-disjoint first-members). Direction A = corpus-unattested stratum is **not** “ghost lemmas” by default (FINDINGS §86).
- **`per-sense-freq`** — frequency per sense ID with an explicit denominator; zero-count senses stay in the table.
- **`genre-strat`** — same join, stratified by corpus genre/register; refuse a pooled number when strata diverge hard.
- **`timeline`** — lemma/entry coverage over edition or scan date when the dict side has a time axis (csl-atlas ribbon shape; H1484 coverage ribbon).
- **`definition-typology`** — classify definition *form* across dictionaries (synonym-gloss vs equivalence vs encyclopaedic, …) with a gold sample + accuracy report before bulk apply (H1483). The typology is a **dictionary-side inventory**, then joinable to corpus axes above.
- **`sense-recon`** — pilot cross-dict sense reconstruction / aligned sense view over a bounded sample (H1587); refuse full-dict claims without the pilot residue table.
- **`wsd-two-witness`** — two-witness word-sense disambiguation / sense-frequency with an independent second witness (H1588); report agreement and both denominators.

## Phase 4 — Honest lower bounds

Where the corpus is lacunar (untagged stems, sandhi-hidden forms, missing genre labels):

- Publish a **lower bound**, not a confident zero.
- State what the corpus does **not** tag (the SanGram EM-limit pattern).
- Seed a small validation sample (≈50 tokens/loci) and report false-positive count in the output.

Never put a live LLM in the measurement path — numbers must be recomputable by a third party (`/claim-verify` / `/dict-comparative-test` bar).

## Phase 5 — Deliverables

One session = one join:

1. Committed join TSV (or parquet if the repo already uses it) with provenance header comment or sibling `.meta.json` (pins, key, denominators, n).
2. Coverage report Markdown with both denominators, residue classes, and the validation-sample FP count.
3. Optional viz only via `/viz-page` if a site surface already exists for this repo — do not invent a one-off dashboard here.
4. Register the dataset in the kosha manifest and/or `PROJECT_INTERLINKS.md` when the join is reusable.
5. CHANGELOG `[Unreleased]` bullet in the owning repo → `/cut-release` when the repo versions.

## Hard rules

- Dict×dict key reconciliation → `/crosswalk-build`, not this skill.
- Descriptive census of one committed layer with no join → `/census-stat`.
- Graded-reader layer over a passage → `/reader-layer`.
- Detector column on an existing crosswalk → `/crosswalk-build` detector-column mode.
- Never fabricate attestation from model “knowledge” of the language — only from the committed corpus feed.

## Report

One line: pair, axis, `|∩|/|dict|`, `|∩|/|corpus|`, residue sizes, pin/SHA, report path.

_Dr. Mārcis Gasūns_
