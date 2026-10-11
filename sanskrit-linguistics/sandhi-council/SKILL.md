---
name: "sandhi-council"
description: 'Adjudicate an AMBIGUOUS sandhi junction by multi-agent council - independent methods + corpus-frequency views propose split/rule. Triggers: "sandhi-council", "this sandhi is ambiguous". Opus 4.8 (claude-opus-4-8)/Fable 5 (claude-fable-5) tier. Do not use for routine unambiguous splitting or checking — use /sandhi-split or /sandhi-check; reserve this for junctions where independent methods genuinely disagree.'

---

# sandhi-council

Use this skill when the user asks to run the migrated source command `sandhi-council`.

## Command Template


# /sandhi-council <ambiguous string | junction>

_Created: 14-07-2026 · Last updated: 05-09-2026_

For the hard cases the deterministic tools flag but can't settle — a sandhied string with
multiple valid resolutions (e.g. `tavedam` = `tava idam` or `tava edam`?). Built on the
sandhi programme ([SANDHI_PROGRAMME.md](https://github.com/gasyoun/kosha/blob/main/SANDHI_PROGRAMME.md)).

## Procedure

1. **Convene independent perspectives** — **four real spawns of the `sandhi-voice` subagent, one lens each, in a single parallel batch.** "No peeking at the others" is only true with genuine context isolation; four personas prompted in this one context are four rephrasings of a single opinion, which is the defect this delegation fixes. Each returns `LENS` / `SPLIT` / `RULE` / `CONFIDENCE` / `BASIS` / `ALTERNATIVES`, and `no reading` is a real contribution. The four lenses:
   - **Gold/corpus voice** — is this string (or its shape) attested in DCS? what does `corpus_sandhi.tsv` say the likely rule is?
   - **Neural voice** — the DharmaMitra method-C segmentation.
   - **FST voice** — the vidyut-cheda segmentation.
   - **Grammar voice** — Pāṇinian expectation for this environment (the rule that *should* apply), independent of frequency.
2. **Judge (Opus 4.8 / Fable 5).** Adjudicate on the evidence: agreement across voices, corpus frequency of each candidate rule, and the grammar. Where the voices disagree, the disagreement IS the finding — state what makes it ambiguous.
3. **Verdict** — the adjudicated split + rule, a confidence, and the dissent recorded. If genuinely undecidable without context, say so and name what context would resolve it (metre, semantics, the surrounding text).

**Guardrail:** the council is for GENUINE ambiguity — don't convene it on a junction the deterministic tools already resolve confidently (that's `/sandhi-split`). And frequency is evidence, not proof: the rarer resolution can be the right one in context.

_Dr. Mārcis Gasūns_
