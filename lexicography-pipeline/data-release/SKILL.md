---
name: "data-release"
description: 'Package a derived dataset as a FAIR, citable release — publish-safety gate first (rights uncertainty is NOT a stop), provenance README, versioned release, Zenodo DOI + CITATION.cff round-trip. Triggers: "release the dataset", "make X citable".'
---

# data-release

Use this skill when the user asks to run the migrated source command `data-release`.

## Command Template


# /data-release

_Created: 03-07-2026 · Last updated: 26-09-2026 (H5498: mechanical route ported)_

## Default path (seconds — run this first)

```sh
python3 ~/Documents/GitHub/Uprava/tools/data_release.py --dir .                    # report + planned gh/zenodo commands (dry-run, read-only)
python3 ~/Documents/GitHub/Uprava/tools/data_release.py --dir . --bump patch       # plan the version+date rewrite
python3 ~/Documents/GitHub/Uprava/tools/data_release.py --dir . --bump minor --apply   # rewrite CITATION.cff version + date-released ONLY
```

The script checks CITATION.cff (version / date-released / title / doi / cff-version, malformed values fail loud), checks `.zenodo.json` presence + title/description/license, computes the bump math, and PRINTS the `gh release create` + Zenodo draft commands (token read at runtime, never printed) — it never runs them; `--apply` edits only the CITATION.cff version + date lines. `--selftest` checks it offline.

Papers here increasingly stand on derived data (Whitney crosswalk triples, corpus_lexicon, frequency layers, the Nikitin LOD spine) — and `/paper-submission-pack` covers papers, nothing covers making the DATA citable. A dataset nobody can cite is a dead deliverable. This is `/publish-safety-check`'s constructive sibling: gate first, then package.

**Standing rights policy (github-wide, MG 30-07-2026):** rights uncertainty is **not** a stop condition, including for publication. Record known rights facts once; do not park the release on a missing “license ruling.” Canonical: [STANDING_POLICY_RIGHTS_UNCERTAINTY_IS_NOT_A_STOP_2026.md](https://github.com/gasyoun/Uprava/blob/main/docs/STANDING_POLICY_RIGHTS_UNCERTAINTY_IS_NOT_A_STOP_2026.md) · FINDINGS §255. Keep the wording **risk-tolerant** — do not soften it in the pack notes.

LLM / judgment residue only after the script report exists — the metadata battery and version-bump math are mechanical; freeze-readiness, datasheet and rights wording come after it, by hand.

Argument: the asset (name, path, or the PROJECT_INTERLINKS feed entry).

## Phase 0 — Safety gate (blocking only for real block classes)

Run `/publish-safety-check` on the owning repo/asset first. NO-GO only for confirmed prohibition, explicit restricted-data designation, privacy exposure, secrets, intentional private-only visibility, or bulk-leakage — **not** for unclear rights. Record any **known** inherited constraint once (e.g. confirmed Zaliznyak CC BY-NC on commercial packaging; scoped Sementsov clearances) and proceed with an honest license field.

## Phase 1 — Resolve the asset

From `Uprava/PROJECT_INTERLINKS.md` / `SHARED_CODE.md` §8/§10 or the repo itself: exact files, the generating pipeline, upstream inputs (each with ITS license), whether the asset is regenerable-and-gitignored (then the release must ship a built copy — a pointer to a gitignored path is not a release), and current row-counts/coverage figures verified against the actual files, not remembered ones.

## Phase 2 — FAIR pack

In the owning repo:

- **Datasheet** — every release ships a filled Gebru-style datasheet (motivation, composition,
  source edition/page range where applicable, encoding + transliteration regime, collection
  process, known gaps & label-quality state, license, intended use, maintenance) before
  publishing. Reuse csl-observatory's generalized
  [`docs/DATASHEET_TEMPLATE.md`](https://github.com/sanskrit-lexicon/csl-observatory/blob/main/docs/DATASHEET_TEMPLATE.md)
  rather than authoring the section skeleton from scratch each time. **No derived dataset
  release without a filled datasheet.**
- **README** for the asset (dated header + byline, full blob URLs): what it is, schema (columns/fields with one example record), verified counts, how it was derived (pipeline + inputs + model tier and exact version for any LLM-derived layer), known caveats (pull the matching FINDINGS.md entries — the ⚠️s ARE the documentation), and how to cite.
- **License** — data license (CC BY 4.0 default when that is the real choice; CC BY-NC where **confirmed** inheritance requires it; `unknown` / `mixed` / quoted upstream text when unclear) distinct from the repo's code license; put it in the README and a `LICENSE-DATA` file if the repo's LICENSE is code-only. **Ambiguity is not a park:** ship the pack with the honest field; do not invent a human `@DECIDE` solely because the license is incomplete.
- **CITATION.cff** — dataset type, byline per the identity memory (no "Dr.", ORCID `0000-0003-4513-884X`), version, repo URL; validate with `cffconvert --validate` if available.
- **Version + release** — changelog entry, then `/cut-release` in the same pass (that rule is standing). The release carries the built artifact when the data is gitignored-regenerable.

## Phase 3 — Zenodo DOI + CITATION.cff round-trip at freeze

At the version freeze, run the full round-trip, not just a pending-marker:

- **Reserve BOTH DOIs** — Zenodo issues a **concept DOI** (all versions, the one papers should usually cite) and a **version DOI** (this release). Where the Zenodo–GitHub integration is already enabled for the repo, the version DOI mints on the release webhook and the concept DOI is stable across releases; record both slots explicitly.
- **Sync `CITATION.cff` in the same pass**: `version:` matches the tag being cut, `date-released:` the freeze date (ISO here — CFF spec, not the DD-MM-YYYY header rule), `doi:` the concept DOI (version DOI in the release notes), byline per the standing identity memory — `Mārcis Gasūns`, no "Dr.", ORCID `0000-0003-4513-884X`. Validate with `cffconvert --validate` if available.
- **Bump release metadata**: the GitHub release notes and the asset README cite the same version + DOI pair — three places (README, CITATION.cff, release notes), one truth.
- **Two mint paths — do not confuse them.** The GitHub↔Zenodo *software* hook archives `v*` tags and does **not** mint a dataset DOI (H2685: SanskritLexicography concept `10.5281/zenodo.21306715` is software). A derived four-file pack needs a **classic** deposit at `https://zenodo.org/api/deposit/depositions`.
- **Classic dataset mint is agent-doable when the PAT file exists.** Read `~/.secrets/solyanka.env`, take the line that **starts** `Zenodo:` (case-insensitive), strip the prefix, use that token as `Authorization: Bearer …`. Never print or commit the token. Never use any other line in that file. Catalog of paths: [Uprava/docs/SECRETS_INDEX.md](https://github.com/gasyoun/Uprava/blob/main/docs/SECRETS_INDEX.md). PUT file uploads need `Content-Type: application/octet-stream` (a missing type returns 415). After publish, verify the record **logged-out** via `https://zenodo.org/api/records/<id>` (OBS-T false-DOI class: a reserved id that 404s is not a mint).
- **Human `@DO` only if the file or the `Zenodo:` line is missing** — then the `@DO` is "create `solyanka.env` with a `Zenodo:` PAT line", not "log into zenodo.org and click upload". Do not leave `DOI: pending` when the file is sitting on disk.
- If the repo's GitHub↔Zenodo hook is the intended *software* DOI, still do not wait for the webhook (`/cut-release` Phase 3b). A dataset still gets its **own** concept DOI.

## Phase 4 — Register + report

- `Uprava/PROJECT_INTERLINKS.md`: the asset's feed entry gains released-version + license + DOI-status (or a new entry if it was never registered — that's a `/findings-append` case too).
- **`kosha/data/manifest/datasets.json`** — add or update this asset's row in the same pass (mandatory per the [github-spine CLAUDE.md](https://github.com/gasyoun/github-spine/blob/main/CLAUDE.md) spine table): public tier ships as a [`data-v*` release](https://github.com/gasyoun/kosha/releases), restricted tier stays private per [`DATA_HUB_ROADMAP.md`](https://github.com/gasyoun/kosha/blob/main/DATA_HUB_ROADMAP.md). kosha is the data-hub that actually serves the bytes; PROJECT_INTERLINKS only records the dependency, it doesn't distribute.
- `Uprava/ARTICLES.md`: link the dataset from any paper that stands on it.
- Commit/PR per repo rules. Report: release link, license ruling, DOI @DO row, hub rows touched (including the kosha manifest row), and any caveat the README now carries that the consuming paper must echo.

## Gate that stays (LLM + human)

The script checks metadata bytes — it never answers whether the dataset is actually FROZEN for release: the publish-safety Phase 0 NO-GO classes, a filled datasheet, honest rights/license wording, and the three-places-one-truth sync (README · CITATION.cff · release notes). Green metadata ≠ release-readiness.

_Dr. Mārcis Gasūns_
