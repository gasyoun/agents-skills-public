---
name: "venue-scout"
description: 'Shortlist 4–6 live, in-scope publication venues for one paper from Uprava/ARTICLES.md — liveness checked against recent issues, scope vs TOCs, OA policy/fees compared. Sonnet 5 (claude-sonnet-5) / Opus 4.8 (claude-opus-4-8) / Fable 5 (claude-fable-5) tier, web-heavy. Triggers: "where to submit Axx", "venue for the paper", "find a journal".'

---

# venue-scout

Use this skill when the user asks to run the migrated source command `venue-scout`.

## Command Template


# /venue-scout

_Created: 03-07-2026 · Last updated: 05-09-2026_

A21 sat on HOLD because its venue was picked without scouting: one target journal turned out defunct since 2019, the fallback out of scope. With ~15 papers in the pipeline, venue selection recurs — and every venue claim here must be verified live, not remembered, because journal landscapes rot fast.

Argument: the stable paper ID (`Axx` / `Mxx`) from [`Uprava/ARTICLES.md`](https://github.com/gasyoun/Uprava/blob/main/ARTICLES.md).

## Phase 1 — Paper profile

From `ARTICLES.md` + the manuscript: field (lexicography / Indology / DH / corpus linguistics), type (research article, data paper, review), language of the manuscript, length, and any constraints already ruled (a locked venue decision is not re-litigated — check decision docs and memory first). Data papers change the candidate set entirely (JOHD, Research Data Journal, etc.) — classify honestly.

## Phase 2 — Candidate research (web)

Find 6–10 candidates, then verify each before it may appear in the shortlist:

- **Alive** — a published issue within the last ~12 months, on the journal's OWN site (aggregator pages lie about dead journals — the A21 lesson). Defunct/dormant → note-and-exclude, don't shortlist with a caveat.
- **Scope** — read the actual recent tables of contents, not the mission statement; a paper like ours in the last 2 years is the strongest signal.
- **The practicals** — OA policy + APC (fees matter — an independent scholar has no institutional deal), peer-review turnaround if stated, indexing (Scopus/WoS/ERIH PLUS), submission format + portal URL, language(s) accepted.

**Fan the candidates out to the read-only `venue-liveness-verifier` subagent — one venue per spawn, in parallel.** Each returns `LIVENESS` / `SCOPE FIT` / APC / OA / turnaround / indexing / portal with the URL behind every field, plus an `EXCLUDE?` line; fetching 6–10 journal sites and their tables of contents so that one shortlist row survives is exactly the context this session must not inherit. Every fact in the shortlist carries the URL it came from. The comparison table, the ranking and the recommendation stay here — the agent verifies one venue and does not compare venues.

## Phase 3 — Shortlist memo

`<Axx>_venues_<DD-MM-YYYY>.md` next to the manuscript (dated header, byline, full blob URLs): a comparison table of the 4–6 survivors (fit / OA+fee / turnaround / indexing / liveness evidence), the excluded-with-reasons list (stops future sessions re-proposing the dead ones), and a ranked recommendation with one paragraph of why.

## Phase 4 — Hand off the choice

The pick is MG's: run [`/decision-record`](https://github.com/gasyoun/claude-config/blob/main/commands/decision-record.md) for it if the session is interactive, otherwise add a @DECIDE to [`Uprava/GTD_NEXT_ACTIONS.md`](https://github.com/gasyoun/Uprava/blob/main/GTD_NEXT_ACTIONS.md) linking the memo. Once ruled, write the venue into the paper's `ARTICLES.md` row. Commit per repo rules. Report: shortlist table inline, recommendation, where the decision now lives.

_Dr. Mārcis Gasūns_
