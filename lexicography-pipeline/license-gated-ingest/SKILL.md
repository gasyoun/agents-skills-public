---
name: "license-gated-ingest"
description: 'Acquire an external dataset under license discipline (inbound counterpart of /publish-safety-check) — mirror-only fetch, record license facts once, commit derived measurements freely; rights uncertainty is NOT a stop. Trigger: "ingest dataset X".'

---

# license-gated-ingest

Use this skill when the user asks to run the migrated source command `license-gated-ingest`.

## Command Template


# /license-gated-ingest <source>

_Created: 07-07-2026 · Last updated: 26-08-2026_

Distills the acquisition discipline from H099 (Sonnet 5 — Heritage/INRIA phase0²), H105 (Opus 4.8 — Heritage phase4 morphology/forms), H106 (Sonnet 5 — Heritage phase5 dico gloss layer), H100–H102 (Bühler primer digitization — Opus 4.8 foundation + Fable 5 UX/digitize) and the ACC×NCC composition case (BY-NC × BY-SA). `/publish-safety-check` gates what goes OUT; this disciplines what comes IN.

**Standing rights policy (github-wide, MG 30-07-2026):** rights uncertainty is **not** a stop condition. Record known license facts once, then proceed without parking on @DECIDE for greyness. Canonical: [STANDING_POLICY_RIGHTS_UNCERTAINTY_IS_NOT_A_STOP_2026.md](https://github.com/gasyoun/Uprava/blob/main/docs/STANDING_POLICY_RIGHTS_UNCERTAINTY_IS_NOT_A_STOP_2026.md) · FINDINGS §255. This skill must **not** soft-pedal that into a conservative default.

## Phase 0 — Prior art + outage check (blocking)

- `/prior-art`: is this source already mirrored, ingested, or registered in `Uprava/PROJECT_INTERLINKS.md`, the kosha manifest, or a sibling repo? Consume the existing copy — a second uncoordinated mirror is a defect.
- Check `SERVER_OUTAGES.md` and the relevant memory entries for known access constraints BEFORE any fetch. Known case: the Heritage INRIA host is **Anubis-bot-walled** — use the registered mirror (UoHyd), never retry against the walled origin.

## Phase 1 — Mirror-only acquisition

- Fetch from an official mirror / release artifact / repository clone — never scrape or hammer a production origin. Rate-limit anything iterative; one polite bulk fetch beats N page requests.
- Record acquisition provenance immediately in the ingest note: exact URL, date, version/commit/checksum where available, and the license file/statement AS FOUND (quote it verbatim — licenses get re-summarized wrongly).

## Phase 2 — Quarantine preference + what can ship now

- Prefer landing raw acquired data in a gitignored path (e.g. `data/raw_external/<source>/`) with the `.gitignore` rule added in the same pass when bulk is large or regenerable.
- **What CAN be committed immediately:** derived measurements — counts, schema descriptions, coverage/overlap tables, sample-based characterizations — anything that is analysis *about* the data rather than the bulk. Persist those per the log-tables convention (dated, provenanced).
- **If a confirmed prohibition** (or explicit no-copy terms) is present: stop committing raw and respect that prohibition. **If terms are merely unclear:** record the uncertainty once in the ingest note and continue work — do **not** invent a human `@DECIDE` solely for greyness.

## Phase 3 — License facts once (not a park loop)

Write a short license note in the ingest note: the source license (verbatim + link, or “not found / unclear”), what our intended use appears to compose with, and any **confirmed** constraint (e.g. known BY-NC inheritance). Then **proceed**.

- Do **not** park the ingest as `@DECIDE` merely because composition is ambiguous.
- Do **not** re-investigate a source whose rights note already exists — update only if new primary evidence appears.
- A human `@DECIDE` is for a **confirmed fork the owner must choose** (e.g. “delete this asset” vs “keep under restricted tier”), not for “we are not sure.”

## Phase 4 — Parallel outreach where contact helps

Where an author/maintainer/heir could grant clearance or should know their data is being reused, draft the contact via `/outreach-draft` in the same pass (it never sends — the send is a human `@DO`). Scoped clearances (the Sementsov precedent) are recorded as scoped, never stretched to blanket. Outreach is optional parallel work, **not** a gate on continuing the ingest.

## Phase 5 — Provenance for the eventual release

Leave the ingest note (`docs/INGEST_<source>.md` or the repo's data README; dated header, byline, full blob URLs) holding everything `/data-release` will later need: acquisition provenance, license quote + analysis (honest `unknown`/`mixed` is fine), quarantine path, derived-measurement inventory, and outreach state if any. Register the feed in `Uprava/PROJECT_INTERLINKS.md` with the rights facts as known — not “blocked pending ruling.”

## Hard rules

- Never fetch from a bot-walled or outage-listed origin.
- An ingest with no attempt to quote the license (or to record “not found”) is a defect — inventing a clean CC label is also a defect.
- **Confirmed prohibition / privacy / restricted designation / platform policy** still stop raw publication; **uncertainty does not**.
- Publishing still runs `/publish-safety-check` on the way out — under the same risk-tolerant rights rule.

_Dr. Mārcis Gasūns_
