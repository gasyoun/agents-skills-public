---
name: "dh-memo"
description: 'Author a repo-anchored ACL/DH research-agenda memo — ≥6 falsifiable hypotheses citing nearest existing IDs, ≥5 viz specs, ACL-Anthology method crosswalk, ranked backlog. Memo-only, no build. Triggers: "research agenda for repo", "DH memo".'

---

# dh-memo

Use this skill when the user asks to run the migrated source command `dh-memo`.

## Command Template


# /dh-memo <repo>

_Created: 07-07-2026 · Last updated: 28-08-2026_

Distills the 07-07-2026 research-agenda fleet (H265 seed → H269 csl-observatory → H271 csl-corrections → H272/H278–H282 csl-guides → H273 csl-atlas, the canonical spec). One session = one repo-anchored **design memo + spec** answering the owner's five standing questions. Fable 5 (`claude-fable-5`) for the analysis; Opus 4.8 (`claude-opus-4-8`) fallback.

## The five-question spine (every memo answers all five)

1. What **new hypotheses** are testable at this repo, beyond its existing index?
2. What **visualisations** should be added or upgraded?
3. How can **data from non-Cologne dictionary repos** be reused to raise the repo to best digital-humanities standards — scientifically AND pedagogically?
4. What **new sections** could mine data layers the repo already commits but doesn't surface?
5. What does **aclanthology.org** teach, and how do those methods fold in?

## Phase 0 — Ground truth first

- `/prior-art` reflex: read the repo's hypothesis registry (`docs/HYPOTHESIS_INDEX.md` or its rigor reports), existing memos in `docs/`, `BOUNDARY_RULES.md` / boundary notes, and glob the actual `data/` tree — never trust a brief's summary of what data exists.
- **Verify every reuse asset before proposing on it** — glob the owning repo or the kosha manifest; anything unconfirmable goes to the Deferred appendix as *unverified*, never asserted. State the join key (SLP1 lemma, or dict-siglum × headword).
- Check `.ai_state.md` WIP + GTD for a concurrent session on the same memo.

## Phase 1 — The memo (seven sections, in order)

Target `<repo>/docs/<TOPIC>_MEMO.md` or `docs/<REPO>_RESEARCH_AGENDA.md` (~2,500–4,500 words; depth over padding):

1. **Executive summary** — 3–5 highest-leverage moves, one line each, tagged by priority layer.
2. **New testable hypotheses** — 6–10, numbered with proposed IDs. Each entry: one-sentence claim · committed dataset(s) that test it (+ join key if reuse) · method (statistic/model, with ACL precedent where one exists) · expected figure/output · **research read** · **learner read** · **nearest existing hypothesis ID + delta** · data-readiness. Floor: **≥6 genuinely new, none a reword** — the nearest-ID-plus-delta citation is what proves that. Any hypothesis resting on a human-labeled sample must specify a frozen gold sample + Cohen's κ + P/R/F1.
3. **Visualisation proposals** — **≥5**, numbered. Each: title · new path OR existing page to upgrade (prefer upgrade) · dataset + loader · chart type in the repo's own stack · text/ASCII mockup · research read · learner read · effort S/M/L. Include at least one view over an orphaned/committed-but-unsurfaced dataset and at least one diachronic/timeline view when the data allows.
4. **ACL-Anthology method crosswalk** — a table: method → linked ACL paper(s) → which committed dataset it applies to → what hypothesis/viz/section it enables → nearest thing the repo already does (gap delta); close with a "what to adopt first and why" paragraph. **Live fetch, real research** — never synthesized from memory; every citation a clickable link; reuse the H265 survey's gathered citations where they cover the ground rather than re-fetching. Distinguish "applies to committed data now" from "needs net-new derivation".
5. **New sections mining additional layers** — per layer: what it surfaces · reuse asset(s) + join key · the `Claim → Evidence → Source` chain · derivation/ETL needed and **which repo owns it** (boundary check) · research + learner read.
6. **Prioritised build backlog** — ranked by leverage: rank · leverage · effort · dependencies · owning repo · model tier (Fable 5 (`claude-fable-5`) analysis-heavy / Sonnet 5 (`claude-sonnet-5`) mechanical build / Opus 4.8 (`claude-opus-4-8`) ambiguous judgment) · a ready-to-paste literal starter line per item, `H###` placeholder form (`Read ~/Documents/GitHub\Uprava\handoffs\H###-<Model>_<repo>_<slug>_DD.MM.YY.md and execute it.`) — build handoffs are NOT minted this session.
7. **Deferred / out-of-scope appendix** — boundary-routed items and unverified reuse assets.

## Hard rules

- **Memo-only.** No pipeline runs, no new pages, no `data/` mutation, no half-built anything — every item ends in a spec + a future starter line. (H269: "produce ONE planning document that later build-sessions execute.")
- **Dual audience, both reads mandatory:** every hypothesis/viz/section carries a research read (defensible, ideally ACL/WSC-publishable) AND a learner read; "if an item only serves one audience, either give it the second read or cut it."
- **Claim → Evidence → Source:** every displayable claim traces to a committed artifact, or is flagged "needs data derivation first". No "we could explore…" bullets — every item falsifiable and buildable. No vibes.
- **Boundaries are routing contracts:** proposals that belong to another repo (TEI/OntoLex → csl-standards, corpus/DCS → VisualDCS, content research → csl-atlas, etc.) say so and route there — a mapping contract, not an in-repo build.
- **Consume canonical, never re-derive:** if a proposal touches a dataset with a known canonical source (e.g. MW roots via the kosha release), it names that source — never adds another parallel derivation.
- Doc hygiene per the standing contract: `DD-MM-YYYY` dated header, `_Dr. Mārcis Gasūns_` byline, plain Markdown (no HTML), **full blob/tree URLs for every file reference** — memos get pasted across repos.

## Delivery

Branch + PR (public repos take no direct-main push), merge same pass per the handoff-scoped rule. Mirror any `@DECIDE`/`@DO` the memo surfaces to `Uprava/GTD_NEXT_ACTIONS.md`; register any new feed contract in `PROJECT_INTERLINKS.md`; consider `/findings-append` for durable classifications (e.g. the ACL crosswalk); close the handoff via [`/handoff-close --reconcile`](https://github.com/gasyoun/claude-config/blob/main/commands/handoff-close.md) (H1342 — the row flip + close clause buffer into the session's single `registry_reconcile.py flush`). Stamp model tier + exact version in the memo's provenance line.

_Dr. Mārcis Gasūns_
