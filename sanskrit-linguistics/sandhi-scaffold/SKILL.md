---
name: "sandhi-scaffold"
description: 'Scaffold sandhi rule-extraction on gold-split corpora: junction-rule inducer ("X Y to Z" rules), frequency tables, gold validation, splitter bake-off, pedagogy surfaces (generalized DCS-sandhi). Trigger: "sandhi-scaffold". Do not use for a single text or a one-off check — this builds the rule-extraction programme itself; use /sandhi-check or /sandhi-split once rules already exist.'

---

# sandhi-scaffold

Use this skill when the user asks to run the migrated source command `sandhi-scaffold`.

## Command Template


# /sandhi-scaffold

_Created: 14-07-2026 · Last updated: 05-09-2026_

Generalises the **kosha DCS-sandhi programme** (the reference implementation:
[SANDHI_PROGRAMME.md](https://github.com/gasyoun/kosha/blob/main/SANDHI_PROGRAMME.md),
built [H882](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H882-Opus_kosha_corpus-sandhi-pedagogy-roadmap_13.07.26.md) → [H902](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H902-Opus_kosha_sandhi-phase4-pedagogy-surfaces_14.07.26.md), both Opus 4.8 (`claude-opus-4-8`); released kosha v0.36→v0.47) to **any gold-split corpus**. The
core idea: a corpus token that carries both its **surface** (post-sandhi) form
and its **underlying** (pre-sandhi) form contains an *implicit* sandhi rule at
every word junction — this scaffolds the machinery to make those rules explicit,
frequency-ranked, validated, and teachable.

**Argument:** the target corpus (path) + its gold-split format — where the two
forms live (e.g. DCS CoNLL-U: FORM column + MISC `Unsandhied=`). If unstated,
Phase 0 discovers it.

**Do NOT reinvent the inducer.** The kosha scripts are the tested reference —
adapt them to the new corpus's format, don't rewrite from scratch:
[`dcs_sandhi_induce.py`](https://github.com/gasyoun/kosha/blob/main/scripts/dcs_sandhi_induce.py)
(inducer), [`build_corpus_sandhi.py`](https://github.com/gasyoun/kosha/blob/main/scripts/build_corpus_sandhi.py)
(sweep), [`compare_sandhi_methods.py`](https://github.com/gasyoun/kosha/blob/main/scripts/compare_sandhi_methods.py)
(A/B/C bake-off), [`score_gita_gold.py`](https://github.com/gasyoun/kosha/blob/main/scripts/score_gita_gold.py)
(gold scoring), [`build_sandhi_curriculum.py`](https://github.com/gasyoun/kosha/blob/main/scripts/build_sandhi_curriculum.py) /
[`build_sandhi_reference.py`](https://github.com/gasyoun/kosha/blob/main/scripts/build_sandhi_reference.py)
(pedagogy surfaces).

---

## Phase 0 — find the gold signal (the make-or-break step)

The whole method rests on locating, per token: (1) the **surface** form as it
appears in running text, (2) the **underlying** pre-sandhi form. Confirm both
exist and read a real sample before writing any code.

- Where do they live? (a column, a MISC field, a parallel file, an alignment?)
- **Critical gotcha — merged surfaces / multi-word tokens.** Many corpora record
  vowel-coalescence and compound merges NOT in the per-token surface but in a
  **multi-word-token span** (CoNLL-U range line `n-m`) or a separate continuous
  line. Per-token FORM is often *un-coalesced*, so a token-edge diff MISSES vowel
  sandhi entirely. Find where the merged surface lives before assuming FORM has it.

## Phase 1 — the junction-rule inducer

For each adjacent token pair, diff underlying vs surface at the **junction** and
emit the rule `IN_LEFT IN_RIGHT → OUT`. Four hard-won guards (each cost real
iterations in kosha — bake them in from the start):

1. **Facing-edge guard.** Only read the rule off the FACING edges — the left
   token's END, the right token's START. A token's form can differ from its
   underlying at its *far* edge (sandhi with the OTHER neighbour, or lemma
   normalisation like `me`→`mama`); attributing that to this junction produces
   garbage rules. Require the change to touch the facing edge.
2. **Merged / MWT mode (mode 2).** For a coalesced span (see Phase 0), align the
   merged surface against the concatenated component underlyings and read the
   internal-boundary rule (`na`+`agniḥ` in `nāgnir` → `a a → ā`). Handle the
   span's right edge too (its final sandhi with the token after it).
3. **Spaced vs merged notation.** True vowel coalescence merges into one output
   (`a a → ā`); but when the right token's initial phoneme *survives*, write it
   SPACED (`i e → y e`, `t b → d b`) so it matches a hand-annotated table. Split
   the output when the right phoneme survives; keep merged otherwise.
4. **Phoneme awareness.** Aspirate digraphs (`kh`, `bh`…) are one phoneme;
   consonant clusters (`br`) are not. NFC-normalise on read.

Then **categorise** (rule-class families) + **frequency-rank**; emit per-text +
merged tables (`rule · category · count · pct · n_texts · examples`).

## Phase 2 — validate against gold (if an independent hand rule-set exists)

If a human ever hand-annotated the same rules on one text, score **frequency-MASS
coverage** (not rule-string count): Σ gold-count over shared rules ÷ Σ gold-count.
kosha's inducer hit **96.3 %** this way on the Bhagavadgītā. A small pilot
under-attests rare rules — score on the text the gold actually covers. Missing
mass points straight at the specific rule classes still to fix.

## Phase 3 — splitter A/B/C bake-off (for texts with NO gold split)

For the no-gold path (e.g. GRETIL, raw e-texts), the inducer needs a splitter.
Compare, feeding the SAME inducer so only the splitter differs, scored against
gold on shared texts (mode-1 F1):

- **A** gold splits — the ceiling. **B** a rule-based/FST splitter (offline).
  **C** a neural segmenter (API).
- **kosha finding, likely to generalise:** neural (C) ≫ rule-based (B) —
  F1 ~0.75 vs ~0.28, precision 0.9+. **Default to the neural splitter for the
  no-gold path**, but re-measure on the new corpus.

## Phase 4 — pedagogy surfaces (optional, if teaching is a goal)

Consume the merged table: **graded curriculum** (order by
`frequency × class × environment-generality`, weights in a *tunable config*, not
hard-coded — "learn N rules → read X %"); **per-class reference**; **drills**
(split/join/identify items + Anki + web quiz); **reader hover** (rule + frequency
on a reader). Theme-aware, self-contained pages.

## Phase 5 — register + ship + propagate

- Manifest / dataset registry row per new table; changelog → cut a release.
- Cross-repo: name the **consumers** in the interlinking hub and the feature
  catalogue so siblings don't re-derive it (kosha did this via
  `Uprava/PROJECT_INTERLINKS.md` + `SanskritLexicography/FEATURES_INDEX.md`).
- Author a `SANDHI_PROGRAMME.md`-style hub doc; mint handoffs for deferred surfaces.

---

**Scope note.** This scaffolds the *rule-extraction* programme; it does not itself
build a novel splitter or a full learning app — those are the bake-off's method-C
API and Phase-4 surfaces respectively. Lean on the kosha reference; adapt the
format-specific reader (Phase 0/1 step 1), keep the four guards intact.

_Dr. Mārcis Gasūns_
