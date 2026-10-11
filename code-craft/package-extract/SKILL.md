---
name: "package-extract"
description: 'Extract a reusable core into a shared package; migrate consumers behind golden-diff parity gates. Migration Sonnet 5 (claude-sonnet-5); boundary calls Fable 5 (claude-fable-5)/Opus 4.8 (claude-opus-4-8). Trigger: "extract this into a package".'

---

# package-extract

Use this skill when the user asks to run the migrated source command `package-extract`.

## Command Template


# /package-extract <source-repo/module> [consumer-repos...]

_Created: 11-07-2026 · Last updated: 18-09-2026_

Distills the extraction discipline run three times (H077 Fable 5 — extract; H103 Fable 5 — migrate behind a golden-diff gate; H120 Fable 5 — Wave-3 pilots ×3 repos in one day): census candidate C5 in [FABLE_FOOTPRINT_SKILLS_CENSUS.md](https://github.com/gasyoun/Uprava/blob/main/archive/FABLE_FOOTPRINT_SKILLS_CENSUS.md), built via [H581](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H581-Fable_Uprava_package-extract-skill-build_11.07.26.md). The `sanskrit-util` family (see [SHARED_CODE.md](https://github.com/gasyoun/github-spine/blob/main/SHARED_CODE.md)) is the standing precedent: one canonical source per code family, consumers adopt behind a parity gate, never a big-bang rewrite.

Arguments: the module to extract; optionally the consumer repos to migrate this pass (default: pilot ONE, list the rest as queued ports).

## Phase 0 — Prior art (mandatory)

`/prior-art` first: does a canonical owner for this family already exist in SHARED_CODE / sanskrit-util / a sibling repo? If yes, this session is a *migration to it*, not a new package. Also decide the extraction boundary here (what goes in, what stays consumer-local) — the one genuine judgment call; record it in the package README.

## Phase 1 — Extract with the tests it already has

Move the core into the package (own repo or the established shared one) WITH its existing tests + golden vectors; add none of the "while we're here" improvements — behavior changes and extraction never share a commit. Package gets: README with the boundary decision, version, CHANGELOG seed (the `/cut-release` reflex applies on every future changelog edit).

## Phase 2 — Golden-diff parity gate

Before any consumer switches: run the consumer's real workload over a **pinned corpus** (declared: which inputs, how many, which commit pins them) through OLD path and NEW package; outputs must be **byte-identical** (or the diff explicitly ruled acceptable, each hunk justified in the gate record). Commit the gate record (corpus id, checksums/diff-zero evidence) next to the migration PR — the H103 pattern. No parity record → no switch.

## Phase 3 — Pilot adoption, git-pinned

Migrate ONE consumer first, pinning the package by git tag/commit (never a floating branch). PR per consumer repo, auto-merge per the large-work rule. Remaining consumers become queued ports: one GTD row each with the parity-gate recipe linked (the H120 wave shape — several pilots can run same-day once the recipe is proven).

## Phase 4 — Register

SHARED_CODE.md gains/updates the family row (canonical source = the package) in the same pass — the post-work hub sync rule; `/findings-append` any measured migration gotcha. Model tier + exact version everywhere.

## Hard rules

- No consumer switches without a committed byte-parity record on a pinned corpus.
- Extraction commits change zero behavior; improvements ship after adoption, as package releases.
- Consumers pin by tag/commit; a floating dependency is a defect.

## Mode — intra-repo decompose (U64, skill-mine H4542–H5153 vote 18-09-2026)

Same discipline, **no new package**: one repo's module has outgrown its seams and the call sites drift. Invoked as `--mode intra-repo` (or when the argument is a single module inside one repo).

1. **Name the seams by domain, not by line count** — the split targets cohesive responsibilities (routes by domain, one tool's concerns, one shared primitive), never an arbitrary "under N lines" chop. Record the seam list before touching code.
2. **Extract/move WITH the tests the code already has.** No "while we're here" improvements — behaviour change and decomposition never share a commit (the Phase 1 rule, unchanged).
3. **Consolidate drifted duplicates to ONE canon inside the repo** — the 13-reinventions case (`tools/lockutil.py`), the near-duplicate mirror case; the canonical file gets the shared util, call sites migrate onto it.
4. **Migrate every call site in the same pass** — a half-migrated split is worse than no split.
5. **Behaviour-parity record in the same pass** — the intra-repo analogue of the Phase 2 golden-diff gate: `route:list --json` byte-identical before/after, test-suite parity, `php artisan route:list` / CLI `--help` / fixture outputs unchanged; "zero behavior change" is stated in the commit and proven by the record, not asserted.
6. **No SHARED_CODE row, no version bump** — nothing left the repo. If the extracted core turns out to be wanted by a second repo, THAT becomes an ordinary cross-repo extraction (Phases 0–4 above) with its own parity gate.

Evidence (13 executed + 3 minted, 15–17-09-2026): H4956 (`tools/_cli.py` shared subprocess/argparse/retry core + call-site migration) · H4957 (typed-exception + logging convention migration) · H4948 (55-function namespace → cohesive modules) · H4962 (`tools/git_refs.py` extracted) · H4941 (`routes/web.php` 1326 lines → 12 domain files) · H4952 (`Kernel.php` → 6 concern traits) · H4978 (StudentController domain split) · H5003 · H5004 (mint_handoff.py decomposition finished) · H4947 · H4961 · H4881 · H4860 (split re-ported with `route:list` parity). Related: [`/simplify`](https://github.com/gasyoun/claude-config/blob/main/commands/simplify.md) (clarity-only, no behaviour contract) · [`/package-extract`](https://github.com/gasyoun/claude-config/blob/main/commands/package-extract.md) Phases 0–4 (cross-repo).

_Dr. Mārcis Gasūns_
