---
name: "critical-apparatus"
description: 'Critical-edition variant apparatus: two-witness LCS, Gotoh akshara loci, pilot-batch rollout, print verification, regression-gated regen. Sonnet 5 (claude-sonnet-5); method gates Opus 4.8 (claude-opus-4-8)/Fable 5 (claude-fable-5).'

---

# critical-apparatus

Use this skill when the user asks to run the migrated source command `critical-apparatus`.

## Command Template


# /critical-apparatus

_Created: 17-07-2026 · Last updated: 29-08-2026_

The edition-apparatus discipline proven on the Rāmāyaṇa Sundarakāṇḍa (critical ↔ southern) and the
full 18-parva Mahābhārata (BORI critical ↔ Nīlakaṇṭha vulgate, 298,930 loci): a **2-witness
collation producing its own derived variant apparatus** — positional loci with competing readings
(`crit ] vulg`), not opaque per-verse similarity scores. Home repo:
[`CommentaryStrategies`](https://github.com/gasyoun/CommentaryStrategies); rollout ledger:
[`docs/EDITION_APPARATUS_ROLLOUT.md`](https://github.com/gasyoun/CommentaryStrategies/blob/main/docs/EDITION_APPARATUS_ROLLOUT.md).

**Core rule — the pipeline is layered, and the layers never bleed:**
1. **Coarse book-level bucketing** (identical / variant / critical-only / witness-only / transposition)
   via cheap difflib-LCS + fuzzy rescue — owns structure, renumbering, absences.
2. **Fine akṣara-level apparatus** (Gotoh affine-gap over akṣara tokens, near-equivalence substitution
   matrix) — operates one level finer, *inside* a matched pair only. **Never touch the book-level
   comparator when upgrading the aligner** (H776 rule); its totals are the standing regression gate.
3. **Human gate** — footnote candidates reach an interactive HTML review sheet emitted by the shared
   `render_review_sheet(items, config)` ([`csl-pyutil`](https://github.com/sanskrit-lexicon/csl-pyutil)
   v0.3.0/v0.3.1, the [/review-sheet](https://github.com/gasyoun/claude-config/blob/main/commands/review-sheet.md) V1–V8 standard) — never markdown checkboxes, never
   hand-rolled sheet HTML; no apparatus is cited in scholarly output before philologist review.

**Prior art (run `/prior-art` before writing ANY new Python — every driver exists):**
aligner [`scripts/spike_helayo_align.py`](https://github.com/gasyoun/CommentaryStrategies/blob/main/scripts/spike_helayo_align.py)
(char-level Gotoh + `syllabify()`/`gotoh_aksara()`/`collapse_loci_aksara()`) · Rāmāyaṇa comparator
[`scripts/compare_editions.py`](https://github.com/gasyoun/CommentaryStrategies/blob/main/scripts/compare_editions.py) ·
MBh comparator [`scripts/compare_editions_mbh.py`](https://github.com/gasyoun/CommentaryStrategies/blob/main/scripts/compare_editions_mbh.py)
(parva-parameterized; ISO-15919 + Devanagari loaders) · apparatus builder
[`scripts/build_edition_apparatus.py`](https://github.com/gasyoun/CommentaryStrategies/blob/main/scripts/build_edition_apparatus.py)
(generalized CLI: `--input --outdir --title --work-label --other-key --chapter-label`) · batch driver
[`scripts/run_all_mbh_parvas.py`](https://github.com/gasyoun/CommentaryStrategies/blob/main/scripts/run_all_mbh_parvas.py)
(programmatic per-book README + .gitignore) · print verifier
[`scripts/verify_mbh_apparatus_against_print.py`](https://github.com/gasyoun/CommentaryStrategies/blob/main/scripts/verify_mbh_apparatus_against_print.py) ·
footnote wiring [`scripts/build_edition_footnotes.py`](https://github.com/gasyoun/CommentaryStrategies/blob/main/scripts/build_edition_footnotes.py)
(`variant_reading` candidate kind) + [`scripts/build_footnotes_review_html.py`](https://github.com/gasyoun/CommentaryStrategies/blob/main/scripts/build_footnotes_review_html.py)
— that last one **must emit through `render_review_sheet()`, not its own markup**: hand-rolled sheet HTML
is banned for new sheets, so port it to the emitter (item shape
`{id, filt, title, badges, question, panels, note_placeholder}` + `title_href`; `context`/`links` are not
fields) rather than extending its template.
The alignment method is **helayo** (Charles Li's Sanskrit MSA engine — see org memory
`reference_helayo_sanskrit_alignment`): **consume the method, don't rebuild it** — the house
implementation is a stdlib Gotoh reimplementation (H776 option b), deliberately preferred over
executing the upstream binary (which would need a trusted-binary + publish-safety check first).

## Arguments

The work + witness pair (or a repo already carrying them), then a mode:
`spike` (new witness pair — method fit check + adopt gate) · `pilot <book>` (one kāṇḍa/parva
end-to-end) · `rollout [books…]` (batch the rest) · `print-verify` (App.-I-style independent check) ·
`regenerate` (rerun after an aligner change, regression-gated). Default when unstated: continue the
rollout ledger's next open item.

## Phase 0 — Witness inventory, premise check, rights

- **Verify the witnesses are actually distinct editions before building anything** (H783 premise
  failure: GRETIL's TEI "southern recension" was the *same* critical edition as the already-held
  text, just TEI-formatted — identical sarga counts). Cross-check verse counts against an
  independent census where one exists; a matching total is the standing sanity gate (H784/H802
  matched `NILAKANTHA_VULGATE_CENSUS.md` exactly; H804's two off-by-231/-1 parvas were fully
  explained by the census's own documented empty-`mula` service-record caveat — chase every delta
  to a documented cause, never shrug it off).
- **Rights first:** prefer already-cleared sources (the valmikiramayan.net → Gita Supersite CC BY 4.0
  rebuild at H783 was a prior-art miss that forced redoing 5 comparisons). Bulk witness text stays
  **gitignored, local-only, regenerable**; provenance doc per source (BORI_CRITICAL_SOURCE.md shape).
  Committed artifacts = aggregates, short excerpted loci, prose READMEs only. `git add -n` before
  every commit to prove no bulk-text file is staged. Publication beyond the repo is gated by
  [`/publish-safety-check`](https://github.com/gasyoun/claude-config/blob/main/commands/publish-safety-check.md).
- Isolated worktree off `origin/main` — CommentaryStrategies is contention-guarded and multi-actor.

## Phase 1 — Book-level comparison (coarse)

Parameterized comparator (`compare_editions.py` / `compare_editions_mbh.py` / a new sibling for a new
work — parameterize, don't fork): normalize both witnesses at **load time** (Devanagari → IAST via
`sa_align.deva_to_iast`; ISO-15919 vocalic `r̥`/`l̥`/`r̥̄`/`l̥̄` → precomposed IAST `ṛ`/`ḷ`/`ṝ`/`ḹ` —
see gotcha #2), then difflib-LCS content alignment + fuzzy-match rescue on
**character-4-gram Jaccard, never word tokens** (see gotcha #1). Output: identical / variant pairs /
critical-only / witness-only (split into true structural absence vs reworded) / transpositions,
contiguous witness-only runs (footnote candidates), whole extra chapters — with BOTH witnesses'
verse ids in the `book.chapter.verse` shape.

## Phase 2 — Akṣara-level apparatus (fine)

`build_edition_apparatus.py` over the clean-variant pairs: akṣara segmentation (maximal-onset IAST
syllabification), Gotoh affine-gap alignment over akṣara tokens with the char-level near-equivalence
matrix (ā~a, ṃ~m, ś~ṣ, n~ṇ …) nested as the substitution cost between two akṣara strings; collapse
into positional loci; final merge step also merges consecutive loci sharing an identical non-empty
string on ONE side (gotcha #3). Heavily-reworded pairs route to the absence layer, not the apparatus.
Spot-check ≥3 variant pairs by hand per book — genuine textual variants, not alignment noise — before
declaring a run good.

## Phase 3 — Pilot, then batch rollout

Never run all books blind. **Pilot one book** (pick one with independently-known material for
cross-checks), verify census counts + spot-checks, commit, THEN batch the rest **smallest-first**
(surfaces bugs cheaply before the largest books — H804 order). The batch driver writes each book's
README + `.gitignore` programmatically from the run's own JSON, one subdirectory per book. Update the
rollout ledger doc in the same pass.

## Phase 4 — Independent print verification

"Verify against print" without a physical book: the critical edition's own **apparatus criticus in
electronic form** (BORI App. I star passages, `bombay.indology.info .../apps/`) is the real print
apparatus, not a reconstruction. Match every structural-absence flag (+ a sample per extra chapter)
via 4-gram character Jaccard on fully **despaced** canon strings with an inverted n-gram index
(~1–50s/book; plain `SequenceMatcher` is O(n·m) per pair and died at corpus scale — gotcha #4).
Calibrate the threshold by manual spot-checks across score bands before trusting it. **A ~20%
confirmation rate is the expected, meaningful outcome, not a defect** — the edition's apparatus was
never exhaustive over later vulgate accretions; ~1 in 5 independently attested is a real positive
signal the absence-detector finds recension differences rather than alignment noise. Committed
verification artifacts carry verse ids + scores + matched-passage ids ONLY — no verbatim text.

## Phase 5 — Regression-gated regeneration

Any aligner/normalizer change (H776 akṣara upgrade, H830 fixes) reruns the apparatus ONLY under a
regression gate: (a) book-level totals reproduce the committed numbers **exactly** (the coarse layer
must be untouched by construction); (b) the already-shipped apparatus of the reference book
(Sundarakāṇḍa) is re-verified — same clean-variant verse set, no verse's locus count increases, each
fix provably only merges/reduces. **If the new run moves in the *opposite* direction from the
reference result** (H830: MBh loci doubled where Sundarakāṇḍa had shrunk 21%), STOP and root-cause
before shipping — that instinct found two real shared-code bugs. A residual gap that survives the
fixes (H830's 2.1×) gets sampled, characterized honestly, and **flagged as an open question for a
human ruling — never silently shipped as "done and clean."**

## Gotchas (each cost a real session)

1. **Word-token Jaccard fails on sandhi-fused text** (H783 rebuild, H810 first attempt: 3.9%
   confirmation): witnesses disagree on compound spacing, silently collapsing match rates
   (Yuddhakāṇḍa ~100/5700 matched vs ~5000 with char n-grams on identical input). Always
   char-4-gram Jaccard on despaced strings.
2. **ISO-15919 vocalic r/l never NFC-unifies with precomposed IAST** (H830 fix 2): `r̥` (r +
   U+0325) has no canonical equivalence to `ṛ` (U+1E5B) — without an explicit map, every vocalic-r
   word (~1000+/parva) becomes a pure encoding-noise locus (`'dr̥ḍhaiḥ' ] 'dṛḍhaiḥ'`).
3. **Witnesses disagree on whitespace inside sandhi-joined compounds** (H830 fix 1: `varām̐l
   labdhvā` vs `varāṃllabdhvā`) — the space-short-circuit rule fragments one locus into two
   pointing at the same word; merge consecutive loci sharing one identical side (provably
   count-reducing only).
4. **`SequenceMatcher.ratio()` at corpus scale is a trap** (H810): >20 min on one parva even with a
   length prefilter — Sanskrit verses cluster too tightly in length to discriminate. Inverted
   n-gram index instead.
5. **Adjacent insertion/deletion next to a substitution fragments loci at char level** (H776 5.3.11
   / 5.3.19 cases) — the reason akṣara-level is the production mode; char mode is spike-only.
6. **Don't parse the apparatus criticus's anchor-comment prose** (heavily line-wrapped, fragile) —
   group star-passage lines by passage id and match on text content (H810).
7. **A "new" witness may be an old witness in a new format** (H783 GRETIL TEI) — premise-check
   before scoping a rollout on it.
8. **Center-Star MSA needs ≥3 witnesses** — with 2 its multi-way advantage is latent; re-run when
   the 3rd witness (e.g. Gita Press) is digitised, don't build for it early.

## Dedupe

[`/commentary-apparatus-ingest`](https://github.com/gasyoun/claude-config/blob/main/commands/commentary-apparatus-ingest.md) ingests human commentator notes
into the Sundara apparatus (LLM in the middle, register-gated) — it does not generate edition
variants. This skill owns the **generation** half: deterministic witness collation → loci. They meet
at `build_edition_footnotes.py`, where `variant_reading` candidates join the same review gate — one
emitter-generated sheet on the [/review-sheet](https://github.com/gasyoun/claude-config/blob/main/commands/review-sheet.md) standard, named
`commentarystrategies-edition-apparatus_<book>_review.html` + `..._decisions.json` (a generic
`review.html`/`decisions.json` is banned), with the three verbs approve · reject · defer, the mandatory
`show_ids` / `note_min_height_px` / `save_as` config keys, per-item `title_href`, and the v0.3.1
dark-theme contrast fix; a requested rewrite is a note on a reject, never a fourth button.

## Stop conditions

Stop and report when: the witnesses turn out to be the same edition (Phase 0); rights for a witness
are unresolved and no cleared substitute exists; a regression gate fails; a regenerate run moves
opposite the reference direction and the root cause is not found; or new machinery beyond the
existing script family seems required for a first-time witness pair — that is an adopt-gate spike
(H776 shape: spike doc + a human ADOPT ruling before production wiring).

## Reporting

Books run · book-level totals (+ census cross-check result) · clean-variant pairs → loci per book ·
absence/reworded split · spot-check verdicts · print-verification confirmation rates (if run) ·
regression-gate results (if regenerate) · rights posture confirmation (`git add -n` clean) · what
still needs the human gate. Model tier + exact version per run.

_Mirrors the Codex skill `critical-apparatus` (kept in sync via [`/claude-codex-skill-porter`](https://github.com/gasyoun/claude-config/blob/main/commands/claude-codex-skill-porter.md))._

_Skill authored by Fable 5 (`claude-fable-5`), 17-07-2026, H1112 (census #3, evidence H776 H783 H784 H802 H804 H810 H830)._

_Dr. Mārcis Gasūns_
