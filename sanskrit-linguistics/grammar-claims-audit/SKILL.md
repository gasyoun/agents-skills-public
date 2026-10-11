---
name: "grammar-claims-audit"
description: 'Build/extend a textbook claim register: harvest claims into claims.yml, verify via verify_*_dcs.py corpus probes, dual verdict_fact x pedagogy. Trigger: "audit the claims in textbook". Verdicts Fable 5 (claude-fable-5), fallback Opus 4.8 (claude-opus-4-8).'

---

# grammar-claims-audit

Use this skill when the user asks to run the migrated source command `grammar-claims-audit`.

## Command Template


# /grammar-claims-audit

_Created: 17-07-2026 · Last updated: 26-08-2026_

The SanskritGrammar claim-register discipline (Bühler H797 (Fable 5), Zaliznyak-Ocherk HK series, Whitney H1051/H1055 (Sonnet 5 / Opus 4.8)), generalized: a textbook's statements about the language are FALSIFIABLE CLAIMS, and each gets a corpus verdict on two independent axes — is it factually true (verdict_fact), and is it still worth teaching as stated (verdict_pedagogy). The axes never collapse: a claim can be factually loose yet pedagogically sound (Whitney's "most roots" idioms) or exactly true yet pedagogically dead.

Argument: the textbook/chapter (or its repo path) + optionally a claim-ID range; resolve the register location from the repo's existing `claims.yml` if one exists.

Evidence base: H768 (Opus 4.8 — Kochergina claim-verification roadmap) · H773 (Opus 4.8 — grammar asserts corpus denies study) · H797 (Fable 5 — claim-verification backlog verify-and-cross-grammar-generalise) · H800 (Fable 5 — gradation metalanguage register / Zaliznyak 3-work comparison) · H1051 (Sonnet 5 — Zaliznyak 1978 voice hypothesis DCS) · H1053 (Opus 4.8 — imperfect narrative-switching hypothesis) · H1055 (Opus 4.8 — Apte syntax claim-verification seed) — 7 sessions, SanskritGrammar. Dedupe: `/grammar-rule-encode` encodes rules as data, it does not audit truth; `/hypothesis-probe` runs ONE numbered hypothesis — this skill owns the register lifecycle around many claims.

## Phase 0 — Register + contention check

- Locate or create `claims.yml` next to the textbook's working dir. One claim = stable append-only ID (per-source prefix: `BU-`, `HB-`, `HK-`, `ZA-`, …), source quote (original language), locus (page/§), formalization (what would make it false), status.
- **Claim-lock hygiene (FINDINGS §86 — hit on THIS exact shape, H797):** the execution claim-lock is advisory; a concurrent session can ship a slice of the same register anyway. `git fetch` + re-read the register immediately before every push, not just at session start. If a rival slice landed, reconcile via `/dual-run-salvage` (absorb smaller into larger under one ID scheme, never silently pick).

## Phase 1 — Harvest claims

Read the chapter and extract claims, not paraphrase: a claim is a statement the corpus could contradict ("the imperfect is the narrative tense", "class VI roots are rare in the Veda"). Skip definitions and notational conventions. Record each verbatim quote + locus. IDs are append-only — never renumber, never reuse a retired ID.

## Phase 2 — Preregister, then probe

- **Preregister before running** (H1053 precedent, Opus 4.8): for a claim with real inferential weight, write the expected outcome + test design into the register (or QUESTIONS_LOG `[T]` row) BEFORE the probe runs.
- One committed `verify_<claim>_dcs.py` per claim family (`sys.stdout.reconfigure(encoding='utf-8')`, no BOM, next to the register — the numbers must be recomputable). Probes are produce-not-decide: they emit counts/tables; the verdict is ruled in Phase 3.
- **Anchored patterns only** — the Renou classifier defect (unanchored regex stems, ≥7.1% wrong; see `project_renou_classifier_defect`) is the canonical trap of this exact probe genre.
- **Watch the denominator** (quantifier-register lesson, H800, Fable 5): a "9× density" difference can be a denominator artifact. State the denominator in the probe output.

## Phase 3 — Two-axis verdict

Per claim, rule both axes with one line of reasoning each:

- `verdict_fact`: подтверждено / частично / опровергнуто / непроверяемо (say WHY unverifiable — data gap ≠ false; cf. the DCS gaṇa-absence kill-gate, H980-class, Opus 4.8 — saṅgrām W2 thematic-present pilot).
- `verdict_pedagogy`: teach-as-stated / teach-with-caveat / retire. Ruled independently of fact — the reasoning must say what the caveat is.

Verdicts land in `claims.yml`; the register MD is GENERATED from it (computed, never hand-edited — the H689 derive-don't-store rule, Fable 5). Wire probe outcomes to `Uprava/QUESTIONS_LOG.md` (`[T]` rows) in the same pass.

## Phase 4 — Surface + deliver

- Regenerate the register MD; if the book has a reading site, refresh the claim overlay (verdict badges at the loci) and build-verify.
- Branch + PR + auto-merge (worktree if the repo is contention-guarded); update handoff registry + GTD; `/findings-append` any non-obvious corpus gotcha the probes surfaced.
- Model provenance per verdict batch: tier AND exact version.

_Dr. Mārcis Gasūns_
