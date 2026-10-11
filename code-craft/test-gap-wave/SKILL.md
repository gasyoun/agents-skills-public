---
name: "test-gap-wave"
description: 'Close test gaps in ONE wave: census unregistered/unwired test surfaces from the manifest gate red → author offline tests (no network, no secrets, fixture pins) → register in build_test_manifest --check → green gate; surface-bound residuals seamed separately with a named owner. Triggers: "test-gate is red", "close the test gaps", "unregistered test modules". Mechanical — OxAlpha tier.'

---

# test-gap-wave

Use this skill when the user asks to run the migrated source command `test-gap-wave`.

## Command Template


# /test-gap-wave [scope]

_Created: 07-09-2026 · Last updated: 07-09-2026_

Distills the four-session test-gap programme of 06-09-2026: the red-gate fix that registered 6 unregistered modules + stdin receipt hooks ([H4202](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H4202-OxAlpha_Uprava_test-manifest-unregistered-gate-red-stdin-receipt_06.09.26.md)), wave 1 pair-tests for gate scripts ([H4226](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H4226-OxAlpha_Uprava_test-gap-wave1-red-clusters_06.09.26.md)), waves 2–3 selftest split + offline pins, 12/12 modules 247 tests ([H4228](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H4228-OxAlpha_Uprava_test-gap-waves23-wire-and-author_06.09.26.md)), and wave 4 seaming the 17 surface-bound residuals ([H4240](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H4240-OxAlpha_Uprava_test-gap-wave4-seam-bound-residuals_06.09.26.md)). Approved **C48** in the skill-mine delta census ([SKILL_CANDIDATES_CENSUS_H4060_H4304_07_09_2026.md](https://github.com/gasyoun/Uprava/blob/main/SKILL_CANDIDATES_CENSUS_H4060_H4304_07_09_2026.md)), MG chat vote 5/5 approve 07-09-2026.

The discipline this owns: **a test that cannot run offline is a liability, and a gap nobody registered will be re-discovered every red run.**

## Phase 0 — Census the gaps

- Run the manifest check (`python3 tools/build_test_manifest.py --check` on Uprava) and collect the red: unregistered modules, wired-but-failing surfaces, selftests not split.
- Classify each gap: (a) offline-closeable now, (b) surface-bound (needs a real DB/port/handler — seam it), (c) not worth testing (say why, once).

## Phase 1 — Author offline tests

- No network, no secrets, no shared mutable state: fixture pins, tmpdirs, deterministic seeds. A test needing prod creds is a (b) — seam it, never fake-credential it.
- Pair-tests for gate scripts: one positive (gate passes) + one negative (gate refuses) — mirrors `/red-gate-build` Phase 2.
- Register every new module in the manifest in the SAME commit — an authored-but-unregistered test is the H4202 red all over again.

## Phase 2 — Green the gate, honestly

- `build_test_manifest.py --check` green; count modules/tests in the close report (12/12 modules, 247 tests — H4228 shape).
- Skips carry named reasons; an unexplained skip is a reopened gap.

## Phase 3 — Seam the surface-bound residue

- Surface-bound residuals get a separate wave with the seam named (which handler/port/DB, who owns wiring it, when it runs) — H4240: "offline pins only", the seam list recorded, never silently dropped.

## Dedupe fence

- [`/hygiene-track`](https://github.com/gasyoun/claude-config/blob/main/commands/hygiene-track.md) — lint/secrets/md tracks; this skill owns test coverage only.
- [`/eval-harness`](https://github.com/gasyoun/claude-config/blob/main/commands/eval-harness.md) — model-quality benchmarking, not repo test coverage.
- [`/red-gate-build`](https://github.com/gasyoun/claude-config/blob/main/commands/red-gate-build.md) — builds enforcement gates; this skill closes coverage gaps (the manifest gate is its customer).
