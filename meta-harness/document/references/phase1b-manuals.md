
Manuals go stale silently: a feature ships, the manual that governs it does not move.
The generated register ([Uprava/MANUALS_REGISTER.md](https://github.com/gasyoun/Uprava/blob/main/MANUALS_REGISTER.md), built by `Uprava/tools/manuals_register.py`) tracks every estate manual via the `Last updated` date vs the max commit date of its `<!-- governed: … -->` paths (declared in the manual itself). Two tiers: tier A (panel guides) is enforced by coverage tests in CI — never duplicated here; tier B is this pass's job.

1. **Affected-manuals probe (every shipping session):** `python <GitHub-root>/Uprava/tools/manuals_register.py --changed-from <base>..HEAD --repo <home-repo>` — every `AFFECTED:` line is a manual whose governed paths your diff touches.
2. For each affected manual: update the stale section **in the same PR** (reversibility badges where the guide format demands them; run the governing coverage test when one exists), or mint a GTD `@DO` residual through the guarded flow naming the exact stale section — the script itself never writes to stores (unguarded-writer red line).
3. **Full-estate check (batch/drain sessions, or when the register looks suspicious):** `python …/manuals_register.py --check` — exit 1 + STALE rows; same disposition per row.
4. If your artifact ALSO ships a new manual: declare its governed paths (the `<!-- governed: … -->` line next to `Last updated`) and the right tier marker, then regenerate the register — Phase 1's `manual` surface handles registration; this phase handles staleness.

