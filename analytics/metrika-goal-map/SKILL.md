---
name: "metrika-goal-map"
description: 'Wire ONE Yandex-Metrika analytics goal-map contour: counter/goal census via management API (token-gated) → goal naming convention + tiers on live emitters → create goals via API → doc rows beside the feature → 152-FZ counter guardrails (webvisor OFF for logged-in students / guests-only shop) → reachGoal bridge → residual on live dashboard verification. Triggers: "metrika goal map", "цели метрики", "wire analytics goals", "create Metrika goals". Mechanical + judgment — OxAlpha tier; goal taxonomy is judgment.'

---

# metrika-goal-map

Use this skill when the user asks to run the migrated source command `metrika-goal-map`.

## Command Template


# /metrika-goal-map <surface>

_Created: 18-09-2026 · Last updated: 18-09-2026_

Distills the Metrika goal-map line: ORS-FAQ counter tag + `?nc=` ladder suffixes with goal creation parked on the missing token ([H1156](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1156-Sonnet_ORS-FAQ_metrika-tag-nc-suffixes-faq-tree_17.07.26.md)), the Sales UTM convention + goal map ([H1997](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1997-Grok_ORS-FAQ_drain-skill-and-sales-utm_31.07.26.md), PR [#126](https://github.com/gasyoun/ORS-FAQ/pull/126)), the mid-funnel goal-map repair cut as 1.29.1 (PR [#146](https://github.com/gasyoun/ORS-FAQ/pull/146)), and the Systema cabinet contour — 22-goal census → 16 goals created via management API (ids 639422183–234, counter 22→38, MG ruling «заводи») + `webvisor:false` counter + `CLIENT_CABINET_EVENTS` reachGoal bridge, DeepSeek verifier PASS ([H5154](https://github.com/gasyoun/Uprava/blob/main/handoffs/H5154-OxAlpha_Systema-Sanscriticum_cabinet-metrika-tagging-webvisor-off-reachgoal-bridge_18.09.26.md), PR [#2705](https://github.com/gasyoun/Systema-Sanscriticum/pull/2705)). Approved **C52** in the skill-mine delta census ([SKILL_CANDIDATES_CENSUS_H5154_H5155_18_09_2026.md](https://github.com/gasyoun/Uprava/blob/main/SKILL_CANDIDATES_CENSUS_H5154_H5155_18_09_2026.md)), MG chat vote 2/2 approve 18-09-2026 (dual-run with H5156's zero-candidate verdict — the vote is the adjudication).

The discipline this owns: **the goal map is a contour, not an event — census before creation, naming before emission, verification as a dated residual.**

## Phase 0 — Token gate + census (never skip)

- Probe the management API token FIRST (read-only `GET /management/v2/counters`). No token → STOP with the exact ask; park the contour as a dated GTD residual (the H1156 shape: «goal creation not run, no credential spent» — never improvise around a missing credential).
- Census the counter: existing goals (ids + names + types), live emitters (which pages/components fire what), the gap between business funnel steps and mapped goals. Canonical goal names are often missing from the UI — read them via API, not the dashboard (H5154 code/UI gap).

## Phase 1 — Naming convention + tiers (judgment, human-gated)

- Encode the goal-naming convention BEFORE creating anything: canonical names (no UI drift), tier per emitter class (payment → activation → renewal in ONE counter beats scattered micro-counters — H5154), UTM/ladder suffix conventions consistent with prior maps (H1997/H1156 `?nc=`).
- Human gate: the goal taxonomy (what counts as a conversion) is an MG ruling — present the proposed goal table, get «заводи» (or edits), then create.

## Phase 2 — Create + wire

- Create goals via management API (`POST /management/v2/counter/<id>/goals`), recording ids back into the doc row.
- Wire emitters: server-side reachGoal bridge where events live in app code (`CLIENT_CABINET_EVENTS` shape), doc rows beside the feature — the map lives next to the code that fires it.
- **152-FZ guardrails (non-negotiable on RU student surfaces):** `webvisor:false` for logged-in students; webvisor guests-only at most; never record session video of identified students. The counter embed and the privacy ruling land in the SAME change or it does not land.

## Phase 3 — Verification residual

- A goal that exists but was never seen firing is unverified — leave a dated GTD residual to check the dashboard after real traffic (24–48 h), with the exact goal ids to eyeball.
- Register the map: repo docs row + (if a new contour) kosha/Uprava registration per the artifact-propagate sweep.

## Dedupe fence

- [`/viz-page`](https://github.com/gasyoun/claude-config/blob/main/commands/viz-page.md) — builds feed-derived VISUALIZATIONS (incl. Metrika-data heatmaps); this skill wires the GOAL/COUNTER contour that produces the data.
- [`/ors-seo`](https://github.com/gasyoun/claude-config/blob/main/commands/ors-seo.md) — SEO/funnel ladder rungs (meta, email capture, GSC); goal maps are analytics plumbing, not a rung.
- [`/systema-case-feature`](https://github.com/gasyoun/claude-config/blob/main/commands/systema-case-feature.md) — one Systema feature; the goal-map contour is cross-repo (ORS-FAQ + Systema) and predates any single feature.
