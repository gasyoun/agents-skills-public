---
name: "sandhi-diff"
description: 'Comparative sandhi - run the junction-rule inducer over TWO corpora/registers/periods; surfaces the rules that DISTINGUISH them (Vedic <-> Classical proven). Triggers: "sandhi-diff", "diachronic sandhi". Sonnet 5 (claude-sonnet-5) tier. Do not use on a single corpus or text — this compares TWO corpora/registers/periods; for one text''s sandhi use /sandhi-check or /sandhi-scaffold.'

---

# sandhi-diff

Use this skill when the user asks to run the migrated source command `sandhi-diff`.

## Command Template


# /sandhi-diff <corpus-A> <corpus-B>

_Created: 14-07-2026 · Last updated: 05-09-2026_

Built on the kosha sandhi programme ([SANDHI_PROGRAMME.md](https://github.com/gasyoun/kosha/blob/main/SANDHI_PROGRAMME.md)) — reuse
[`dcs_sandhi_induce.py`](https://github.com/gasyoun/kosha/blob/main/scripts/dcs_sandhi_induce.py),
don't reinvent. Both corpora need gold splits (or the same method C splitter — see `/sandhi-scaffold` Phase 3).

## Procedure

1. **Induce both.** Run the inducer on A and on B → two `<id>_sandhi.tsv` tables (rule · category · count · pct).
2. **Normalise to shares.** Convert each rule's `count` to a within-corpus % so different corpus sizes are comparable.
3. **Diff.** For every rule, compute `pct_A − pct_B`; also list **present-in-A-only / B-only** rules and the **category-share shift** (e.g. visarga 32 %→48 %). Rank by absolute delta.
4. **Interpret (Opus 4.8 `claude-opus-4-8`).** The biggest deltas are the linguistic story — name the register/period difference (Vedic non-elision, epic formula sandhi, an author's preference). Flag any delta that's a **corpus artifact** (DCS annotation convention, size, genre) vs a **real** sandhi difference.
5. **Emit.** A `sandhi_diff_<A>_<B>.tsv` (rule · pct_A · pct_B · delta · verdict) + a short findings note; if it's a durable result, `/findings-append` it.

**Guardrail:** a share delta is not automatically a linguistic fact — a corpus's `no-gold`/MWT coverage or genre mix can move shares. Separate convention from language before asserting.

_Dr. Mārcis Gasūns_
