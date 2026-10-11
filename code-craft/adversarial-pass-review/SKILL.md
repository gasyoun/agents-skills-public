---
name: "adversarial-pass-review"
description: 'Adversarially review another agent''s completed work-pass - trust nothing, re-execute, re-derive load-bearing numbers, commit a CONFIRMED/PLAUSIBLE/REFUTED ledger. Trigger: "adversarial review". Fable 5 (claude-fable-5), Opus 4.8 (claude-opus-4-8) fallback.'

---

# adversarial-pass-review

Use this skill when the user asks to run the migrated source command `adversarial-pass-review`.

## Command Template


# /adversarial-pass-review <pass: handoff/PR/report>

_Created: 11-07-2026 · Last updated: 18-07-2026_

Distills the review-of-a-pass shape from H118, H233 (csl-apidev SSR-pass review — every SEO_PLAN number re-derived, a MINOR fix shipped separately) and the H071 money-core multi-finder review: census candidate C9 in [FABLE_FOOTPRINT_SKILLS_CENSUS.md](https://github.com/gasyoun/Uprava/blob/main/archive/FABLE_FOOTPRINT_SKILLS_CENSUS.md), built via [H583](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H583-Fable_Uprava_adversarial-pass-review-skill-build_11.07.26.md). The built-in `/code-review` owns a code *diff*; `/paper-referee` owns a *paper*; THIS skill owns an org **work-pass** — the artifacts + claims one agent session left behind.

Argument: the pass under review — its handoff, PR(s), or report doc.

## Phase 1 — Claim inventory

Read the pass's own account (handoff, PR bodies, report) and extract every verifiable claim into a ledger: stated numbers, "all N items processed", "tests green", "byte-identical", "merged", "no regressions". Each claim gets a row BEFORE verification starts — inventing the checklist after reading the code inherits the original session's blind spots.

## Phase 2 — Re-execute, don't re-read

The core discipline: **everything executable gets re-executed** — rebuild the derived file and diff it, re-run the test suite and count, re-run the metric script and compare digits, re-check the PR actually merged. Reading code and nodding is PLAUSIBLE at best; only re-execution (or independent re-derivation from primary data) upgrades to CONFIRMED. Use read-only `Explore` agents for sweeps and the `adversarial-verifier` agent per finding; where a claim can fail in distinct ways, verify through distinct lenses rather than N identical re-checks. Sample-verify bulk items with a DECLARED sample (state n and selection) when full re-execution is genuinely infeasible — and say so in the ledger.

## Phase 3 — The verdict ledger

Commit `docs/REVIEW_<pass>_<DD-MM-YYYY>.md` in the pass's repo: one row per claim — claim → verification method → evidence link → **CONFIRMED / PLAUSIBLE / REFUTED**, plus findings the pass never claimed (omissions, silent scope cuts). Overall verdict: sound / sound-with-corrections / unsound, with the specific rows that drive it. Model tier + exact version for both the reviewed session and the reviewing one.

**Bind every claim to a baseline.** Each row records the **baseline commit SHA** it was verified at and the **sha256 of every artifact** it re-executed/re-derived (the file, the manifest, the store slice). A verdict is therefore `confirmed at baseline <sha>` — not "confirmed" in the abstract. When a bound file later changes, the claim is relabeled **`superseded`** (re-verify against the new baseline), never silently carried forward as still-CONFIRMED — the same drift discipline as the LANG_PARITY sha-pinned verdicts. This makes a stale confirmation impossible to mistake for a live one.

## Phase 4 — Fixes ship separately

Every REFUTED row spawns its own fix: a separate PR (never folded into the review commit — the H233 pattern), or a GTD `@DO`/minted handoff when out of scope. An unsound verdict on a pass others already built on → flag the downstream consumers in the same pass (PROJECT_INTERLINKS tells you who).

## Hard rules

- No CONFIRMED without re-execution or independent re-derivation; sampling is declared, never silent.
- The review never edits the reviewed artifacts in place — verdict and fixes are separate deliverables.
- REFUTED findings name the concrete evidence (command run, output, diff), not an opinion; the ledger includes what was NOT verified.
- **Preserve raw evidence, never rewrite history.** The command outputs, diffs, and hashes a verdict rests on are captured verbatim (an evidence appendix / attached logs) and kept — a re-review appends a new dated pass and relabels superseded rows, it never edits or deletes the prior evidence. The audit trail is append-only; overwriting it destroys the very thing an adversarial review exists to leave behind.

_Dr. Mārcis Gasūns_
