---
name: "hypothesis-mint"
description: 'Author/dedupe/outcome-tag hypothesis rows in Uprava/QUESTIONS_LOG.md (claim → data → metric → baseline → test artifact; confirmed/refuted/partial/open), recompute win-rate KPI. Triggers: "log this hypothesis", "mint a hypothesis ID".'

---

# hypothesis-mint

Use this skill when the user asks to run the migrated source command `hypothesis-mint`.

## Command Template


# /hypothesis-mint [repo] [claim…]

_Created: 07-07-2026 · Last updated: 07-07-2026_

The QUESTIONS_LOG discipline distilled from H167 (Opus 5 — Uprava questions-log hypothesis backfill, the original backfill) and the H269/H273/H278 hypothesis idiom. Home: [`Uprava/QUESTIONS_LOG.md`](https://github.com/gasyoun/Uprava/blob/main/QUESTIONS_LOG.md) — the single cross-repo register (Uprava is private). Modes: **mint** (one new row), **resolve** (tag an existing row's outcome), **backfill** (sweep a session/repo for unlogged hypotheses).

## The register's shape

- **Three sections:** `[Q]` ad-hoc assistant questions · `[T]` in-task hypotheses (posed mid-work) · `[R]` formal research hypotheses (tied to an `Axx` paper).
- **ID scheme, date-encoded:** `Q<YY><MM>-<NN>` / `T<YY><MM>-<NN>` / `R<YY><MM>-<NN>` — YYMM = month first raised, NN = zero-padded sequence within that month. IDs are append-only, never reused or renumbered.
- **Row columns:** `[T]`: ID · First raised · Last raised · Repo · Hypothesis (as posed mid-task) · Outcome · Resolved · Notes/evidence. `[R]`: ID · First raised · Last raised · Paper (Axx) · Hypothesis · Status · Resolved · Evidence (links the paper + the closing `Dxx` decision where one exists).
- **Outcome tags:** `confirmed / refuted / partial / open` + Resolved yes/no.
- **Per-repo hypothesis IDs are a different layer:** repo-local IDs (csl-atlas `HYPOTHESIS_INDEX.md` PH/H series, guides GH-n, observatory H-n) live in their repos; a QUESTIONS_LOG `[R]`/`[T]` row *references* them, it never mints them. When both exist, cross-link both ways.

## The idiom — what a row must contain

Every hypothesis states **claim → data → metric → baseline → the committed artifact that would test it**. No hand-waving: a claim with no metric or no committed evidence path gets logged `open` with the missing artifact named, not dressed up as testable.

## Workflow

1. **Dedupe semantically first** — grep the log for the claim's key terms before assigning an ID; a re-raised hypothesis updates *Last raised* on its existing row, never mints a twin.
2. **Mint/resolve the row(s)** in the correct section with the next free NN for the current YYMM. Resolving: set Outcome + Resolved + evidence link (PR, report, figure). **An open hypothesis stays open — don't force a verdict.**
3. **Recompute the KPI:** `python Uprava/tools/build_questions_stats.py` (rewrites the win-rate line + weekly trend buckets — auto-generated blocks, never hand-edit).
4. **Regenerate the dashboard:** `python Uprava/tools/build_questions_dashboard.py` (emits `questions-dashboard/questions.js`; opens from disk; **private/local-only — never enable Pages over it without `/publish-safety-check`**).
5. **Commit with explicit pathspec** (`git commit -- QUESTIONS_LOG.md questions-dashboard/questions.js …`), never a bare `git commit` — Uprava hosts concurrent sessions. Push (Uprava takes direct `main` pushes).

## Backfill mode

Sweep sources in this order, logging only hypotheses that were *actually posed*: the session transcript at hand → the repo's `.ai_state.md` `## 🧠 Dev Notes & Hypotheses` → root-cause handoffs → `ARTICLES.md` claims (→ `[R]`). H167's guardrail verbatim: "Don't invent hypotheses to fill the table — only log ones actually made in a real session/paper … a padded one corrupts the win-rate signal."

## Hard rules

- The win-rate KPI is only meaningful if minting is honest — no padding, no forced verdicts, no retro-fitted claims.
- Repo-local hypothesis registries are cross-linked, never absorbed.
- Doc hygiene: full blob URLs, plain Markdown, model tier + exact version in evidence notes.

_Dr. Mārcis Gasūns_
