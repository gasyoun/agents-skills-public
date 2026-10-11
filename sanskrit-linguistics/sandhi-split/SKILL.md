---
name: "sandhi-split"
description: 'Runtime sandhi splitter — paste a raw sandhied Sanskrit string, get words + induced rule at each junction (DharmaMitra neural method C needs --allow-network). Triggers: "split this sandhi", "what are the words in string". Do not use to check whether an already-segmented author''s text applied sandhi correctly — use /sandhi-check instead; for a junction with several plausible resolutions use /sandhi-council.'

---

# sandhi-split

Use this skill when the user asks to run the migrated source command `sandhi-split`.

## Command Template


# /sandhi-split <sandhied string | file>

_Created: 14-07-2026 · Last updated: 05-09-2026_

Runtime front-end to the programme's segmentation, built on
[SANDHI_PROGRAMME.md](https://github.com/gasyoun/kosha/blob/main/SANDHI_PROGRAMME.md).
The A/B/C bake-off settled the splitter: **DharmaMitra neural (method C)** for no-gold text
(F1 ~0.75 vs vidyut ~0.28) — reuse the caller in
[`compare_sandhi_methods.py`](https://github.com/gasyoun/kosha/blob/main/scripts/compare_sandhi_methods.py)
(`_dm_segment`, `https://dharmamitra.org/api/tagging/`, `mode=unsandhied`).

## Procedure

1. **Segment.** Send the string to the neural splitter (`--allow-network`; cache under `data/sandhi/_cache/`). Get the ordered pre-sandhi words. If the string is a known DCS locus, prefer the gold split.
2. **Induce per junction.** For each adjacent word pair, run `dcs_sandhi_induce.induce_rule` on (predicted underlying, observed surface) → the rule `X Y → Z` at that boundary.
3. **Enrich.** Attach each rule's **corpus frequency + class** from `corpus_sandhi.tsv` ("`aḥ a → o '` — visarga, 1.2 % of corpus, #12").
4. **Output.** The split (`word₁ + word₂ + …`) with the rule + frequency annotated at each junction; optionally a small self-contained HTML if the user wants to share it.

**Guardrail:** the neural splitter over-segments compounds — flag low-confidence junctions rather than asserting; offer the gold split when the input is a corpus locus.

_Dr. Mārcis Gasūns_
