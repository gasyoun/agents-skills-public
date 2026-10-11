---
name: "fake-data-lookalike"
description: '152-FZ-safe data sharing: profile a real dataset (per-column distributions, row counts, types), generate a lookalike of the requested size, verify distributions side-by-side — real rows never leave the machine. The <client> hard-deny stays and is not relaxed. Triggers: "fake data", "lookalike dataset", "synthetic copy", "данные наружу", "обезличь выборку".'
---

# fake-data-lookalike

Use this skill when the user asks to run the migrated source command `fake-data-lookalike`.

## Command Template


# /fake-data-lookalike <real.csv> [N]

_Created: 02-10-2026 · Last updated: 02-10-2026_

Generate a privacy-safe **lookalike** of a real dataset (AI Builders peer review item 1, п.4.1, H5634): same statistical shape, zero real rows. Org rule (github-spine AGENTS.md «Fake-data lookalikes»): any dataset leaving the machine to an LLM or a contractor goes out only as a generated lookalike built from a statistical profile of the real set; real rows never leave. The 152-FZ `<client>*` hard-deny (fences H2641/H3502) is NOT relaxed — those paths are never read, never profiled, never shipped.

## Procedure

1. **Fence check first** — refuse any path matching `<client>` (152-FZ) before reading anything.
2. **Profile the real set** (in place, nothing leaves):
   - per column: dtype, null-rate, n_unique, min/max, and the distribution — histogram (≤20 bins) for numeric, value counts for low-cardinality categorical, length stats + top-k pattern regex for free text;
   - row count, pairwise correlation matrix for numeric columns (keep the sign pattern).
3. **Generate the lookalike** of the requested size N (default = real row count):
   - numeric: sample from the binned histogram (or a fitted parametric family when the histogram is unimodal and simple); preserve correlations via per-bin sampling or a copula when the profile shows strong pairs;
   - categorical: sample empirical value frequencies; free text: generate placeholder strings matching the length/pattern profile — never substrings of real values;
   - identifiers: fresh synthetic ids, no reuse of real ones.
4. **Verify side-by-side** — the artifact MG sees is the comparison, not the claim: real-vs-lookalike table per column (mean/std/min/max or top frequencies, ± tolerance), KS-test or bin-share deltas for numerics, row count. A column whose distribution drifted beyond the tolerance is regenerated, not explained away.
5. **Ship** only the lookalike + the profile summary. State in the delivery: «реальные строки не покидали машину; профиль — агрегаты».

## Demo contract

Demo runs on a small local/toy CSV only — no real data in the demo. Artifacts: profile, lookalike, side-by-side check.

_Гасунс_
