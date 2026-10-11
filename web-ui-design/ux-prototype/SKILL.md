---
name: "ux-prototype"
description: 'Benchmark/spec-driven UX prototype directions (HTML mockups, token-layer reskins), browser-verified, winner left human; --spec/--phase0/--implement-dirs modes. Fable 5 (claude-fable-5), Opus 4.8 (claude-opus-4-8) fallback; wiring Sonnet 5 (claude-sonnet-5). Triggers: "mockup", "design directions", "redesign X", "implement dirs", "landing skins", "benchmark against a named site", "compose the hybrid", "production spec", "instrument before we ship".'

---

# ux-prototype

Use this skill when the user asks to run the migrated source command `ux-prototype`.

## Command Template


# /ux-prototype <surface> [n-directions] [benchmark] [--spec] [--phase0] [--implement-dirs]

_Created: 11-07-2026 · Last updated: 05-09-2026_ (U35 taste + implement-dirs · U51 hub mockup-twins)

Distills the highest-velocity Fable 5 (`claude-fable-5`) shape on record — ~10 sessions: H147, H225 (csl-apidev, 4 directions × 4 pages), H323 (samskrte.ru vs the Arzamas benchmark), then the 11-07-2026 wave (H563 dashboard redesign ×4 directions, H568 custdev, H575 ORS-FAQ, H587 kosha, H589 VisualDCS, H654 BookIndex, H660 IndologyScholars). Census candidate C6 in [FABLE_FOOTPRINT_SKILLS_CENSUS.md](https://github.com/gasyoun/Uprava/blob/main/archive/FABLE_FOOTPRINT_SKILLS_CENSUS.md), built via [H580](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H580-Fable_Uprava_ux-prototype-skill-build_11.07.26.md).

The two follow-on modes come from the Systema student-cabinet series (7 sessions: [H822](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H822-Fable_Systema-Sanscriticum_student-cabinet-custdev-ux-remake_12.07.26.md) evidence-led origin → [H954](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H954-Fable_Systema-Sanscriticum_student-cabinet-mockup2-course-workspace-v2_14.07.26.md)/[H956](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H956-Fable_Systema-Sanscriticum_student-cabinet-mockup3-today-first-coach_15.07.26.md)/[H957](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H957-Fable_Systema-Sanscriticum_student-cabinet-mockup4-learning-library_15.07.26.md)/[H958](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H958-Fable_Systema-Sanscriticum_student-cabinet-mockup5-journey-membership-hub_15.07.26.md) the direction series → [H961](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H961-Fable_Systema-Sanscriticum_student-cabinet-hybrid-production-spec_15.07.26.md) the hybrid production spec → [H962](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H962-Sonnet_Systema-Sanscriticum_student-cabinet-remake-instrumentation-phase_15.07.26.md) Phase 0), census item #22 in [SKILL_CANDIDATES_CENSUS_ALL_EXECUTORS_12_17_07_2026.md](https://github.com/gasyoun/Uprava/blob/main/archive/SKILL_CANDIDATES_CENSUS_ALL_EXECUTORS_12_17_07_2026.md), added via [H1129](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1129-Opus_claude-config_skill-update-ux-prototype_17.07.26.md). They exist because that series proved the base skill stops one step short: it ends at "the winner is a human call", and a real remake then has to travel from a picked mockup to shipped production code — the two passes below are that road, and each is its own handoff.

Arguments: the surface (page/dashboard/landing), how many directions (default 1 focused; H563-style exploration = 4), optionally the benchmark site to tear down first. `--spec` runs Mode B (compose the ruled winner into a production spec), `--phase0` runs Mode C (instrument the current surface first), and `--implement-dirs` runs Mode D (multi-dir live skins after a design packet) — Modes B–D presuppose a finished direction series (or an existing design packet), so they never run in the same pass as Phases 1–4.

## Taste-skill orchestration (do not mega-load)

When Fable redesign mockups use [Leonxlnx/taste-skill](https://github.com/Leonxlnx/taste-skill) (or the local install under `~/.grok/skills` / `~/.claude/skills`), **one primary skill per phase** — never load all aesthetic skills at once (minimalist vs brutalist contradict). Canonical pipeline + conflict table:

[docs/TASTE_SKILL_ORCHESTRATION_FABLE_REDESIGN_PIPELINE_2026.md](https://github.com/gasyoun/Uprava/blob/main/docs/TASTE_SKILL_ORCHESTRATION_FABLE_REDESIGN_PIPELINE_2026.md)
(H1972; skill-mine **U35**, 31-07-2026).

Default implementer after direction is locked: `design-taste-frontend` (v2). Audit-first on redesigns: `redesign-existing-projects`. Image comps only: `imagegen-frontend-web` / `image-to-code`. Org skill **this file** owns house multi-direction mockups + Modes B–D.

## Phase 1 — Contract: spec or benchmark teardown

The prototype needs a contract, not taste. Either **(a)** an existing ui-spec / design-system doc in the repo (read it, quote the constraints you'll honor), or **(b)** a benchmark teardown (the H323 pattern): read the reference site, extract the transferable decisions (type scale, density, color logic, navigation shape) into a short committed teardown note — then design against THAT, never from memory of "good design". For any chart/stat surface, use the house palette-token practice (repo-local `src/palette.css` / `palette-tokens.json`, light+dark — see csl-atlas/csl-observatory; the old external `dataviz` skill is no longer installed). Inventory what exists: current page, its CSS tokens, assets — a reskin over live assets (the H660 token-layer pattern) beats a parallel page when the DOM is sound.

## Phase 2 — Build the direction(s)

- Each direction = **self-contained** static HTML/CSS in the repo's `mockups/` (or the established folder — check precedent first): opens by double-click, no server, no external requests (system fonts, inline or local assets).
- Multiple directions differ on a NAMED axis (density, warmth, dark-first, editorial-vs-app) — four shades of the same idea is one direction, not four.
- Light + dark honored (`prefers-color-scheme` at minimum); responsive down to mobile width; real content from the live surface, never lorem ipsum — fake data hides layout failures.
- Non-destructive: mockups never modify the live page; a token-layer reskin uses head-only overrides (`<base href>` + `:root` tokens) so body markup stays byte-identical.

## Phase 3 — Verify in the browser

Open each mockup in the Browser pane: screenshot per direction (light + dark where built), check console clean, resize for mobile. A mockup that was never rendered is a draft, not a deliverable — attach the screenshots as proof.

## Phase 4 — Hand off, winner stays human

PR per repo (auto-merge per the large-work rule — mockups are non-destructive). PR body: one paragraph per direction (its axis, what it optimizes), screenshots, and the **promotion path** (what one edit adopts the winner into the live page — the H660 pattern). The winner is a human call: GTD `@DECIDE` row with the directions linked, never self-ruled. Model tier + exact version.

## Hub mockup-twins — publish, then re-render per ruling round (U51, 26-08-2026)

Every direction is published to [`gasyoun.github.io`](https://gasyoun.github.io/) as the **viewable twin** the human reviews from chat links (the standing html-links rule); the repo-side `mockups/` folder stays the source of truth. After each human ruling round (the R1–R5 pattern), **re-render ALL directions** — including the untouched ones — so the set stays comparable; a set where some pages carry last round's markup is a stale exhibit, not a direction series. Worked example: the kosha word-page wave ([H3457](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H3457-Fable_kosha_wpage-ux-badge-favorites-scananchors_24.08.26.md) directions a/b/c → [H3480](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H3480-Fable_kosha_wpage-header-rethink-flat-tabs-gloss-switch-ru-render_25.08.26.md) direction d + a/b/c re-renders with RU markup → [H3490](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H3490-Fable_kosha_wpage-ru-bare-slp1-quotations-transliterate_25.08.26.md) bare-SLP1 transliteration re-render; five hub `mockups/` commits including the phone-layout fix round). Census U51 in [SKILL_CANDIDATES_CENSUS_H3396_H3535_25_08_2026.md](https://github.com/gasyoun/Uprava/blob/main/SKILL_CANDIDATES_CENSUS_H3396_H3535_25_08_2026.md).

## Mode B — `--spec`: iterate to a production spec

Runs after the direction series, once a human has ruled. Phases 1–4 produce options; this produces the buildable contract. One pass = one spec.

**The series is ruled, never self-driven.** Each mockup ends in a short interview round (~4 questions) whose outcome is one of: iterate this direction again, open the next direction, or compose the winner. H954 iterated B rather than opening A/C/D purely because the human said so. Every ruling gets a stable cumulative ID (`R21`–`R25`, then `R29.0`–`R29.8`) appended to the ONE directions doc — that register is append-only and later phases cite a ruling ID instead of relitigating it. Each iteration is its own handoff; a series does not become one mega-session.

**The winner is usually a hybrid, and composition is an interview, not taste.** The H961 shape: one direction supplies the chassis, the others supply named organs, and every organ is traced to its source direction («from A: the Today band + homework item; from C: ownership shelves + expiry ribbons; from D: the path inside Progress»). Never blend by preference — ask which organ, from where, into what.

**The spec doc carries all of:** the numbered composition (`R##`); a unified precedence rule wherever transplanted organs collide (H961 had to rule one offer-precedence order for four directions' competing offers — an unresolved collision is a bug the build will hit); page-by-page deltas against the reference mockup, not prose description; the engineering bill; the Phase-0 event schema (Mode C consumes it verbatim); and the Phase 0–4 sequence. Sibling metadoc per the org rule. Then queue Phase 0 as its own handoff and stop.

**The spec PR does not auto-merge** — it is a strategic ruling record, so it stays a human merge. This is the one deliberate exception to the large-work auto-merge rule; the mockup PRs of Phases 1–4 still auto-merge as before.

## Mode C — `--phase0`: instrumentation-first baseline

The R20 gate: **a big-bang remake is unmeasurable without a prior baseline**, so no spec code ships until the CURRENT surface emits the spec's event schema and a **≥2-week** baseline has accumulated. Its own handoff, mechanical tier — Sonnet 5 (`claude-sonnet-5`), not the design tier.

- **Instrument the pre-remake surface, change nothing about it.** No UX changes in this pass; money/entitlement code untouched. Emit only the spec events that have a current-surface equivalent, and **honestly list the ones that have no surface yet** in the readout rather than faking a zero.
- **Event names exactly as the spec wrote them** — they carry into the remake unchanged, and that identity is the whole reason before/after is comparable. Renaming one here silently destroys the baseline it was minted for.
- **Prior art before storage** ([/prior-art](https://github.com/gasyoun/claude-config/blob/main/commands/prior-art.md)): H962 wired every event through the repo's existing `activity_events`/`ActivityTracker` pipeline and added no new storage — and deliberately did NOT double-write the two events already captured elsewhere (`lesson.view.heartbeat`, `cabinet.live.zoom.click`), letting the readout aggregate across sources instead. A second write path for an existing event is a defect, not coverage.
- **First-party only.** Server-side or first-party JS; no third-party tracker without an explicit human ruling. PII per the repo's existing hashing conventions — never names, handles, or message IDs.
- **A readout that proves flow.** An aggregate command or admin page showing nonzero counts *through the real emission paths* (not seeded rows) is the deliverable — an event that was never observed arriving is a draft, the same rule as an unrendered mockup.
- **Record observed values; invent no targets.** The baseline reports what is, and the KPI list belongs to the spec. Targets are a human call, later.

## Mode D — `--implement-dirs`: multi-dir live skins after a design packet

Evidence: konsultaciya redesign **H1966** (Fable design packet) → **H1975–H1978** (shell + dirs A/B/C/D). Skill-mine **U35**.

Runs when a **design packet already exists** (committed directions doc + mockups or comps) and a human ruled **ship several directions at once** (or default + optional skins) rather than a single hybrid winner. Distinct from Mode B (one composed production spec) and from Phases 1–4 (static mockups only).

1. **Config gate:** one env / config key (e.g. `MARATHON_LANDING_VISUAL_VARIANT`) + optional `?skin=` query resolves to `a|b|c|d` with a **documented default** (konsultaciya: B default only).
2. **Shell first:** land shared layout/partials + default skin in one handoff (H1975 shape); sibling dirs are parallel handoffs, not one mega-session.
3. **Live stack:** Blade (or the repo's real templates) + CSS tokens + Playwright visual loop — not only `mockups/`. Watcher-afflicted repos → `/watcher-safe-commit`.
4. **One primary taste skill per implement pass** (see orchestration doc); do not re-open the full direction series unless the human re-rules.
5. **Stop:** each dir PR green + smoke for default skin; remaining dirs stay launchable siblings.

## Hard rules

- No direction without a contract (spec or committed teardown); no deliverable without a rendered screenshot.
- Self-contained, non-destructive, real content — external CDNs/fonts and lorem ipsum are defects.
- The prototype pass never picks the winner and never touches the production page beyond the reversible reskin layer.
- Ruling IDs are append-only and are cited, never re-argued; an organ with no named source direction, or a collision with no precedence rule, is not composed yet.
- No remake code ships ahead of its baseline (R20) — Mode C's ≥2 weeks is a gate, not a guideline; and Mode C never renames a spec event.
- Taste skills: **one primary per phase** — no mega-load of contradictory aesthetics.
- Mode D implements a **ruled multi-dir matrix**; it does not invent a fourth aesthetic mid-flight.

_Dr. Mārcis Gasūns_
