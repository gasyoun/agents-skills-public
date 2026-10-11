---
name: "keyswap-feature"
description: 'Build ONE KeySwap/Sanskrit-Writer product unit (HUD, offline wordlist, autocomplete) with release cut — product lane, not re-vendor. Haiku 4.5 (claude-haiku-4-5) thin units; Sonnet 5 (claude-sonnet-5) v3 architecture. Trigger: "keyswap feature".'

---

# keyswap-feature

Use this skill when the user asks to run the migrated source command `keyswap-feature`.

## Command Template


# /keyswap-feature <unit>

_Created: 24-07-2026 · Last updated: 26-08-2026_

Census C22 ([SKILL_CANDIDATES_CENSUS_H1098_H1610_17_24_07_2026.md](https://github.com/gasyoun/Uprava/blob/main/archive/SKILL_CANDIDATES_CENSUS_H1098_H1610_17_24_07_2026.md), H1617 all-approve). Encodes the **KeySwap product slice** on [sanskrit-util](https://github.com/sanskrit-lexicon/sanskrit-util) that six sessions re-improvised: H1545 (Haiku 4.5, 429 HUD), H1552 (Haiku 4.5, offline wordlist), H1563 (Haiku 4.5, DCS-freq option), H1570 (Haiku 4.5, Writer-scheme landing), H1581 (Sonnet 5, v3 former-skips plugins), H1583 (Sonnet 5, network autocomplete).

**Not this skill:** re-vendoring sanskrit-util into consumers → `/cologne-sanskrit-util-sync`. Package extract → `/package-extract`.

Argument: the unit name or handoff (e.g. `offline-wordlist`, `v3-7-network-autocomplete`).

## Phase 0 — Prior art + branch

- Read KeySwap README / `CHANGELOG` / open issues for the unit; `/prior-art` against sibling plugins.
- Worktree off `origin/main` if the clone is multi-session; never invent a second KeySwap tree.

## Phase 1 — One unit, one flag

- Scope **one** user-visible unit per session (HUD string, offline fallback, opt-in annotation, scheme mode, plugin wave item, autocomplete).
- Feature / option defaults **OFF** or opt-in when network or heavy data is involved (H1563, H1583).
- Prefer offline-first paths; network is a fallback after local fuzzy (H1552 → H1583 order).

## Phase 2 — Implement + tests

- Match existing KeySwap TypeScript/JS patterns; do not rewrite the host app.
- Map external errors to clear HUD text (H1545: Cologne HTTP 429).
- Tests green; no live network in unit tests — mock 429 / autocomplete.
- BOM-free UTF-8; no secrets in commits.

## Phase 3 — Release

- CHANGELOG `[Unreleased]` → `/cut-release` when the package versions.
- Note consumer sync only if a release bumps the package consumers need (`/cologne-sanskrit-util-sync` as a follow-on, not this session's default).
- Close the handoff; model tier + exact version in the report.

## Hard rules

- One unit per session; no silent expansion into the whole v3 roadmap.
- Offline before network; flags default safe.
- Re-vendor is a different skill.

_Dr. Mārcis Gasūns_
