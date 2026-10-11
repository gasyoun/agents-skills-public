
- Haiku is the **optional first tier** — per [HANDOFF_PROMPTS.md](https://github.com/sanskrit-lexicon/MWS/blob/docs-pass/papers/microanalysis/HANDOFF_PROMPTS.md), existing on-disk data suffices for the Sonnet 5 / Opus 4.8 passes if this skill is skipped. The refresh is freshening, not a prerequisite.
- H10 (link-rot) is **audit-only** — never edit prose files from this skill. URL fixes belong to the Sonnet 5 / Opus 4.8 passes.
- H5 must precede H1: renderers read `palette-tokens.json` at import. Reversing the order produces stale-colour figures.
- H1 emits the H4 footer — verifying H4 separately catches renderer regressions (e.g. someone hard-codes a date during a hotfix).
- Adjacent-day re-runs are safe: branches are date-stamped (`haiku-refresh-YYYY-MM-DD`), issues are date-titled, and `mwsissues/haiku_refresh/$REFRESH_DATE/` namespaces the artifacts.

