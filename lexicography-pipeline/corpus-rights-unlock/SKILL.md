---
name: "corpus-rights-unlock"
description: 'Once copyright/licence is proven, flip a rights-gated grey corpus to publishable - un-gitignore exports, register a citable dataset (kosha+Zenodo DOI), rewrite rights metadata, changelog->release. Triggers: "rights cleared for X", "unlock the corpus".'
---

# corpus-rights-unlock

Use this skill when the user asks to run the migrated source command `corpus-rights-unlock`.

## Command Template


# /corpus-rights-unlock

_Created: 14-07-2026 · Last updated: 25-08-2026_

Several corpora are committed to public repos under a **"grey per project ruling"**
status: the aligned text is searchable in-app at verse granularity, but the bulk
exports and the search DB are **gitignored** and not redistributed, because the
translation is in-copyright (e.g. SamudraManthanam's Serebryakov Kathāsaritsāgara
and Grintser Rāmāyaṇa; the НКРЯ TMX/para-XML export bundle). This skill executes
the transition **once the owner has actually cleared the rights** — do not run it
speculatively.

**Argument:** the corpus/slug whose rights are cleared, and the evidence
(copyright proof, or a redistribution licence from the rightsholder — translator's
heirs and/or publisher) + the licence type (e.g. CC BY-NC 4.0).

## Precondition — the rights are genuinely cleared

Confirm ALL of:
1. There is concrete evidence: the owner holds the economic rights, **or** a named
   redistribution licence from the rightsholder covers *this* use.
2. The licence terms are known (commercial vs **non-commercial** — Nauka/academic
   works are usually NC; NC blocks some downstream reuse even after unlock).
3. Attribution requirement is recorded (translators stay named regardless — moral
   rights are not waived by an economic-rights transfer).

If any is unproven, STOP — this stays grey; do not unlock on a "probably fine".

## Gate first

Run **/publish-safety-check** on the corpus. It must pass (visibility, personal
data, secrets, gitignored-bulk leakage). Only proceed on GO.

## Unlock — flip in one pass

1. **Un-gitignore the export bulk** for the slug (НКРЯ para-XML + TMX 1.4b + TSV;
   keep `corpus.db` gitignored if it's a rebuildable artifact, not a release asset).
   Verify the un-ignored files are the intended ones and no unrelated bulk rides along.
2. **Rewrite the rights metadata.** In the work `meta.json`: `rights` → the actual
   licence string (holder, date, licence id); set `needs_review: false` **only**
   after the per-source bibliography is verified. Update any `RIGHTS_TABLE.md`.
3. **Register a citable dataset.** Add a [kosha `datasets.json`](https://github.com/gasyoun/kosha/blob/main/data/manifest/datasets.json)
   row; mint a **Zenodo** concept+version DOI; add a row to
   [FEATURES_INDEX.md](https://github.com/gasyoun/SanskritLexicography/blob/master/FEATURES_INDEX.md).
4. **Ship the downstream export.** If an НКРЯ/ruscorpora submission was holding this
   corpus back (see the repo's НКРЯ roadmap), include it now; publish the export
   artifacts (release asset or bulk download on the public site).
5. **Offer bulk download** on the public site if that is the intended surface.
6. **Changelog → release.** Add a `CHANGELOG.md` `[Unreleased]` entry, then
   **/cut-release** (version + tag + GitHub release; sync CITATION.cff/Zenodo).

## Propagate + record

- **/document** the now-published dataset across the hubs (PROJECT_INTERLINKS,
  DATA_LAYERS_CENSUS, the repo README/cold-start doc).
- Update the corpus's rights doc (e.g. `docs/*RIGHTS*UNLOCK*.md`) to record the
  licence and flip its status from "grey" to "cleared", with the date + evidence.
- If a GTD `@DECIDE`/`@WAITING` row tracked the rights question, close it.

## Scope

This governs **redistribution/publication**, not whether search works — the corpus
already functions in-app while grey. If the licence is **non-commercial**,
commercial reuse (paid products) stays blocked even after this unlock; read the
licence before enabling derivative commercial surfaces.

_Dr. Mārcis Gasūns_
