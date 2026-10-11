---
name: "pwg-de-editorial"
description: 'Drain ONE unit of the PWG German editorial/export programme — OntoLex/TEI export, DE sense×DCS pilot, human-gold cut. Triggers: "pwg-de editorial". Opus 4.8 (claude-opus-4-8)/Fable 5 (claude-fable-5) principles+gold; Sonnet 5 (claude-sonnet-5) UI units. Do not use for the PWG→RU translation lane — that is /pwg-drain; this is the separate PWG German editorial/export programme.'

---

# pwg-de-editorial

Use this skill when the user asks to run the migrated source command `pwg-de-editorial`.

## Command Template


# /pwg-de-editorial <unit>

_Created: 26-07-2026 · Last updated: 05-09-2026_

[H1660](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1660-Sonnet_Uprava_skill-mine-h1611-h1659_26.07.26.md) (Sonnet 5 — skill-mine H1611–H1659) vote **D26 force-skill** (MG: one programme is enough if used enough — 10 sessions H1622/H1624/H1628–H1635, full tiers+titles in the Phase 1 table). Encodes the **PWG DE editorial + public-export ladder** that kept re-minting as separate handoffs.

**Not this skill:** live PWG→RU translation windows → `/pwg-drain` family. Ghost-stock forensics → `/ghost-stock-audit`. Generic sense×corpus join → `/sense-corpus-join` (consume it). Visa apply of voted sheets → `/decisions-apply`.

Argument: the unit (e.g. `manuals-truth-pass`, `edition-diff-ui`, `ontolex-tei`, `human-gold-cut`, `public-sidecar`).

## Phase 0 — Prior art + isolation

- `/prior-art` against SanskritLexicography `RussianTranslation/` + PWG DE docs; do not rebuild OntoLex/TEI stacks that exist.
- Worktree off `origin/master` (guarded repo); claim handoff if multi-session.

## Phase 1 — One unit on the editorial ladder

Pick **one** (do not silently expand):

| Unit class | Precedent |
|---|---|
| Manuals / truth-pass (headless) | [H1622](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1622-Sonnet_SanskritLexicography_pwg-manuals-truth-pass-headless_24.07.26.md) (Sonnet 5 — PWG manuals truth-pass headless) |
| Ordered backlog of German layers | [H1624](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1624-Opus_SanskritLexicography_pwg-german-layers-backlog-ordered_25.07.26.md) (Opus 4.8 — PWG German layers backlog ordered) |
| Review sheet (compound differs, etc.) | [H1628](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1628-Sonnet_SanskritLexicography_pwg-compound-differs-review-sheet_25.07.26.md) (Sonnet 5 — PWG compound-differs review sheet) → `/review-sheet` |
| Graph export (OntoLex / TEI) | [H1629](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1629-Opus_SanskritLexicography_pwg-de-graph-ontolex-tei-export_25.07.26.md) (Opus 4.8 — PWG DE graph OntoLex/TEI export) |
| Citation edges / top-n scan links | [H1630](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1630-Sonnet_SanskritLexicography_pwg-citation-edges-topn-scan-links_25.07.26.md) (Sonnet 5 — PWG citation edges top-n scan links) |
| Edition-diff UI | [H1631](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1631-Sonnet_SanskritLexicography_pwg-edition-diff-ui_25.07.26.md) (Sonnet 5 — PWG edition-diff UI) |
| DE sense × DCS attestation pilot | [H1632](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1632-Opus_SanskritLexicography_pwg-de-sense-dcs-attestation-pilot_25.07.26.md) (Opus 4.8 — PWG DE sense×DCS attestation pilot) → join `/sense-corpus-join` |
| Human gold cut + methods packet | [H1633](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1633-Fable_SanskritLexicography_pwg-human-gold-cut-methods-packet_25.07.26.md) (Fable 5 — PWG human gold cut methods packet) → `/gold-adjudicate` / `/review-sheet` |
| Editorial principles doc | [H1634](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1634-Sonnet_SanskritLexicography_pwg-de-editorial-principles-doc_25.07.26.md) (Sonnet 5 — PWG DE editorial principles doc) |
| Public sidecar + Zenodo | [H1635](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1635-Opus_SanskritLexicography_pwg-public-sidecar-zenodo-release_25.07.26.md) (Opus 4.8 — PWG public sidecar Zenodo release) → `/data-release` + `/publish-safety-check` |

## Phase 2 — Build + gates

- Prefer additive fields and CLIs over store rewrites; BOM-free UTF-8.
- Public artifacts: `/publish-safety-check` before Pages/Zenodo; rights in manifest.
- Gold/human cuts never auto-promote; sheet or decisions.json required.
- Tests or `--check` parity for every derived table; CHANGELOG `[Unreleased]`.

## Phase 3 — Close

- Hub sweep: GTD / FEATURES_INDEX / kosha manifest as needed; handoff close.
- Model tier + exact version in the report.

## Hard rules

- One unit per session.
- Do not collapse DE editorial into `/pwg-drain` (RU live lane) or invent a second graph stack.
- MG policy ([H1660](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1660-Sonnet_Uprava_skill-mine-h1611-h1659_26.07.26.md), Sonnet 5): heavy single-programme traffic **is** skill-worthy — keep encoding recurring units here, do not re-defer as "programme only".

_Dr. Mārcis Gasūns_
