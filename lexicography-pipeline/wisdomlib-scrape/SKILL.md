---
name: "wisdomlib-scrape"
description: 'Run ONE unit of a wisdomlib scrape programme — analysis layers, crosswalks, resume/keep-alive, log-driven optimize — under license-gated ingest discipline. Triggers: wisdomlib scrape, wisdomlib layers, resume keep-alive, scrape estate optimize.'

---

# wisdomlib-scrape

Use this skill when the user asks to run the migrated source command `wisdomlib-scrape`.

## Command Template


# /wisdomlib-scrape <unit>

_Created: 26-07-2026 · Last updated: 27-08-2026_

[H1660 (Sonnet 5) — Uprava skill-mine H1611–H1659](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1660-Sonnet_Uprava_skill-mine-h1611-h1659_26.07.26.md) vote **D27 force-skill** (MG: one estate is enough if used enough — [H1625 (Sonnet 5) analysis layers](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1625-Sonnet_wisdomlib-scrape_analysis-layers-crosswalks-full_25.07.26.md), [H1636 (Sonnet 5) resume/keep-alive](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1636-Sonnet_wisdomlib-scrape_resume-keep-alive-check_25.07.26.md), [H1653 (Fable 5) log-driven optimize](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1653-Fable_wisdomlib-scrape_log-driven-optimize-ask_26.07.26.md)). Encodes the **wisdomlib scrape estate** discipline that was deferred as "one programme" then re-improvised.

**Compose, don't replace:** `/license-gated-ingest` (inbound rights) · `/source-health-probe` (up/down before hammer) · `/crosswalk-build` (peer joins). This skill owns the **estate ladder** and stop conditions.

Argument: unit name (`layers`, `keep-alive`, `optimize-ask`, or handoff).

## Phase 0 — Health + rights

- `/source-health-probe` first; if down/degraded, log `SERVER_OUTAGES.md` and stop — do not scrape into a wall.
- `/license-gated-ingest` posture: raw bulk gitignored until license ruling; commit derived measurements + provenance only.
- Worktree; never re-point origin at a bot-walled host.

## Phase 1 — One unit

| Unit | Deliverable | Precedent |
|---|---|---|
| **layers** | L0–L10 analysis layers + peer crosswalks + Samudra integrate plan | H1625 |
| **keep-alive** | Resume / keep-alive check; idempotent continue from checkpoint | H1636 |
| **optimize-ask** | Log-driven optimize interview + ranked fix list (no silent rewrite of raw) | H1653 |

## Phase 2 — Execute

- Checkpoint files + manifests; re-runnable from last good offset.
- Rate limits: backoff, no parallel hammer of origin.
- Crosswalks use length-preserving keys (`/crosswalk-build`); never NFD-strip.
- Optimize unit: measure from logs first; one change class per pass.

## Phase 3 — Close

- Register new derived assets (kosha / PROJECT_INTERLINKS if durable).
- FINDINGS only for non-obvious scrape gotchas.
- Model tier + exact version.

## Hard rules

- One unit per session; no full-estate silent expansion.
- License-gated raw stays local until `@DECIDE`.
- A second scrape estate (non-wisdomlib) may reuse this skill by treating `<unit>` as estate-agnostic — rename only if a second estate's ladder diverges.

_Dr. Mārcis Gasūns_
