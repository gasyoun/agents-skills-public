---
name: "sandhi-check"
description: 'Sandhi linter, INVERSE of extraction — check authored Sanskrit sandhi junction-by-junction, flag missing/wrong/over-applied sandhi vs corpus-attested rules. Sonnet 5 (claude-sonnet-5), Opus 4.8 (claude-opus-4-8). Triggers: "lint my Sanskrit". Do not use to split unsegmented text (use /sandhi-split) or to adjudicate a genuinely ambiguous junction (use /sandhi-council) — this only validates already-segmented text.'

---

# sandhi-check

Use this skill when the user asks to run the migrated source command `sandhi-check`.

## Command Template


# /sandhi-check <text | file>

_Created: 14-07-2026 · Last updated: 05-09-2026_

Runs the sandhi programme's rules *backwards* as a validator, on
[SANDHI_PROGRAMME.md](https://github.com/gasyoun/kosha/blob/main/SANDHI_PROGRAMME.md) data.

## Procedure

1. **Get the intended words.** From the author's own word boundaries (if they wrote it padapāṭha-ish) or by asking for the intended segmentation; else segment with method C and confirm the intent.
2. **Predict the correct surface** at each junction: look up the expected rule for (left-final, right-initial) in `corpus_sandhi.tsv` (the highest-frequency attested outcome for that environment).
3. **Compare** the author's surface to the predicted one at each junction. Classify: ✅ correct · ⚠️ **not applied** (sandhi omitted where the corpus applies it) · ❌ **wrong** (a different, non-attested outcome) · 🟡 **variant** (an attested-but-rarer outcome — Vedic/optional, not an error).
4. **Report** per junction: the author's form, the expected form, the rule, and the verdict — with the corpus frequency as evidence ("`aḥ a → o a` is attested but `o '` is 60 % — likely intended the elision").

**Guardrail — descriptive, not prescriptive.** The corpus shows what's ATTESTED, and Sanskrit sandhi has legitimate optional/register variants (Vedic non-elision, avagraha optionality). Flag a rarer-but-attested form as a **variant**, never a hard error; reserve ❌ for outcomes the corpus never shows.

_Dr. Mārcis Gasūns_
