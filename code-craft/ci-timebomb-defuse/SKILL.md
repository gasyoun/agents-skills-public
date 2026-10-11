---
name: "ci-timebomb-defuse"
description: 'Defuse ONE CI date-timebomb: a hardcoded date in a fixture/seed crossing a now()-boundary in code (subDays window, month end, catch-up boundary) → deterministic red CI on every PR. Triage the boundary (code vs fixture side) → relative dates with ≥5-day margin or Carbon::setTestNow pin → suite-green proof before/after → FINDINGS class note so blame is not misattributed. Triggers: "time bomb", "date bomb", "CI red every PR", "hardcoded date fixture". Mechanical — Sonnet 5 (claude-sonnet-5) tier.'

---

# ci-timebomb-defuse

Use this skill when the user asks to run the migrated source command `ci-timebomb-defuse`.

## Command Template


# /ci-timebomb-defuse <failing-suite>

_Created: 18-09-2026 · Last updated: 18-09-2026_

Distills the timebomb line: the Aug-31 payroll/safe-withdrawal bomb defused with `Carbon::setTestNow` pins (PR [#2214](https://github.com/gasyoun/Systema-Sanscriticum/pull/2214), Fable 5 `claude-fable-5` co-authored) and the TelegramSupportPeerWindowTest bomb — seed 2026-08-19 10:00 landing exactly on the `now()->subDays(30)` boundary in `activeKnownChatPeers()`, all file dates made relative with ≥5-day margin; a dual-run (#2698/[#2699](https://github.com/gasyoun/Systema-Sanscriticum/pull/2699)) produced two independent root-causes that matched (GTD 0DF, 18-09-2026). The class is documented in FINDINGS §349: absolute-date fixture vs now()-filtering code is a time bomb that misattributes blame. Approved **C53** in the skill-mine delta census ([SKILL_CANDIDATES_CENSUS_H5154_H5155_18_09_2026.md](https://github.com/gasyoun/Uprava/blob/main/SKILL_CANDIDATES_CENSUS_H5154_H5155_18_09_2026.md)), MG chat vote 2/2 approve 18-09-2026.

The discipline this owns: **a deterministic red is a boundary crossing — find the boundary, defuse with margin, prove green both sides.**

## Phase 1 — Triage the boundary

- Reproduce the failure locally at today's date; identify the exact predicate that flips (which `now()`-relative expression, which constant date).
- Decide the side: is the CODE wrongly time-relative (fix the code) or the FIXTURE wrongly absolute (defuse the fixture)? Most bombs are fixtures; a wrong-side fix re-arms the bomb.

## Phase 2 — Defuse with margin

- Fixtures → relative dates anchored on `now()` with an explicit ≥5-day margin comment to the nearest boundary they must not cross.
- Where the test NEEDS a frozen clock (payroll cutoffs, month-end), pin it: `Carbon::setTestNow(...)` (Laravel) / `freezegun` / `vi` fake — pin, never nudge the system clock.
- Sweep the file for sibling bombs in the same change — one boundary crossing usually means the whole fixture family is absolute-dated.

## Phase 3 — Proof + class note

- Suite green BEFORE the fix at the failing date (or on CI main) and AFTER — both receipts in the commit/PR body.
- Note the class in FINDINGS (one row: repo, predicate, fix shape) so the next «why is CI red on every PR» starts from the class, not from zero — and blame lands on the boundary, not on whoever touched the file last.
- If a dual-run happens (two sessions root-causing independently), treat matched root-causes as confirmation and close the duplicate PR superseded (the #2698/#2699 shape).

## Dedupe fence

- [`/red-gate-build`](https://github.com/gasyoun/claude-config/blob/main/commands/red-gate-build.md) — builds a fail-closed gate that REFUSES bad input; this skill defuses an existing time bomb already firing.
- [`/test-gap-wave`](https://github.com/gasyoun/claude-config/blob/main/commands/test-gap-wave.md) — closes coverage gaps against the manifest; this repairs a deterministic red, not a missing test.
- [`/verify-claims`](https://github.com/gasyoun/claude-config/blob/main/commands/verify-claims.md) — re-verifies report claims; unrelated to CI determinism.
