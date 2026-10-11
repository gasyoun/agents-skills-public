## Inventory (local clones under `GitHub/`, census 23-07-2026)

Cologne front-matter index lists **38** dictionary codes. Status of OCR deliverables (`prefaces/` with source pages + consolidated + usually RU):

### Done (committed locally — use as precedents)

| Code | Repo | Source pages (approx.) | Source lang | Notes |
|---|---|---:|---|---|
| PWG | `PWG` | 27 | de | Reference run |
| PW | `PWK` | 5 | de | |
| GRA | `GRA` | 6 | de | |
| SCH | `SCH` | 5 | de | |
| CCS | `CCS` | 6 | de | |
| MD | `MD` | 10 | en | no separate `.en` |
| CAE | `CAE` | 6 | en | |
| BHS | `BHS` | 15 | en | |
| BOR | `BOR` | 36 | en | large |
| INM | `INM` | 9 | en | |
| IEG | `prefaces_ieg` | 10 | en | staging dir |
| SHS | `SHS` | 2 | en | |
| VEI | `VEI` | 16 | en | |
| MW | `MWS` | 29 | en | |
| MWE | `MWS` | 11 | en | lives under `MWS/prefaces/mwepref*` |
| MW72 | `MW72` | 25 | en | |
| WIL | `Wil-YAT` | 6 | en | shared repo with YAT |
| YAT | `Wil-YAT` | 3 | en | |
| BOP | `BOP` | 5 | la | Latin source |
| STC | `STC` | 5 | fr | French source |
| VCP | `VCP` | 7 | sa | Sanskrit + EN/RU |
| KRM | `KRM` | 24 | sa/en | dense |
| MCI | `MCI` | 28 | en | large |

**csl-guides `ocr-prefaces.mdx` historically listed only PWG/PW/GRA/MD/SCH** — refresh that page whenever a new dict finishes (Phase 6d).

### Not yet OCRed / not production (queue)

| Priority | Codes | Why |
|---|---|---|
| **P0 — promote local-complete first** | `GST`, `PGN`, `SNP` | Full OCR+RU+`*_all.*` still under local `GitHub/prefaces_{gst,pgn,snp}/` only — **not on remote**; do not re-OCR until rescue fails |
| **P1 — high-use general lexica** | `AP90`, `AP`, `AE`, `BEN`, `BUR` | Core missing; BUR = stub scripts only |
| **P2 — classical / indigenous** | `SKD`, `ABCH` | SKD has partial EN in csl-guides `skd.mdx` (docx), not full scan OCR |
| **P3 — partial / specialized** | `LAN` (2/19 local), `ACC`, `LRV`, `PE`, `PUI` | LAN has scans+2 pages on disk; PE/PUI stubs |
| **Empty / stalled stubs** | `BUR`, `SKD`, `PE`, `PUI` | dirs exist, no edition pages |

**Deleted worktrees:** unfinished Claude Code runs not copied into `prefaces_*` or a dict repo leave **no remote trace**. Before `all`, also scan local `GitHub/prefaces_*` for residue.

When arg is `all`: skip every row in **Done** (detect: real-repo `prefaces/README.md` + at least one `*pref_all.*`). Prefer promote P0 locals over re-OCR.
| dict code | local repo | dict code | local repo |
|---|---|---|---|
| PWG | `PWG` | MW / MW72 / MWE | `MWS` (or `MW72`) |
| PW | `PWK` | AP90 | `AP90` |
| GRA | `GRA` | AP / AE | `AP` / `ApteES` |
| BUR/BOP/CAE/CCS/BOR/MD/INM/KRM/LRV/MCI/PUI/ACC/BHS | same-named repo | — | — |
