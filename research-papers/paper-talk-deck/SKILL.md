---
name: "paper-talk-deck"
description: 'Scaffold a talk deck or poster from a near-final ARTICLES.md paper - reuse the paper''s committed figures verbatim, GTD @DO rehearsal. Triggers: "talk deck for Axx", "poster for Axx". Fable 5 (claude-fable-5) tier, Opus 4.8 (claude-opus-4-8) fallback. Do not use for print/camera-ready prep — that is /paper-camera-ready or /book-press-prep; this scaffolds a talk deck or poster only.'
---

# paper-talk-deck

Use this skill when the user asks to run the migrated source command `paper-talk-deck`.

## Command Template


# /paper-talk-deck <Axx> <deck|poster>

_Created: 14-07-2026 · Last updated: 05-09-2026_

The paper family carries a paper all the way to the venue: [`/paper-scaffold`](https://github.com/gasyoun/claude-config/blob/main/commands/paper-scaffold.md) → [`/paper-author-pass`](https://github.com/gasyoun/claude-config/blob/main/commands/paper-author-pass.md) → [`/paper-referee`](https://github.com/gasyoun/claude-config/blob/main/commands/paper-referee.md) → [`/paper-revision-response`](https://github.com/gasyoun/claude-config/blob/main/commands/paper-revision-response.md) → [`/paper-submission-pack`](https://github.com/gasyoun/claude-config/blob/main/commands/paper-submission-pack.md) → [`/paper-camera-ready`](https://github.com/gasyoun/claude-config/blob/main/commands/paper-camera-ready.md) — and then **stops**. Nothing turns the finished paper into the thing you stand up and present: a **talk deck** or a **poster**. That is this skill. It is a scaffold, not a substitute for the author: it lays out the structure and drops in the paper's own already-committed figures, then leaves the register-bearing lines (the spoken framing, the one-sentence hook) as stubs for a human to voice.

Arguments: the stable paper ID (`Axx` / `Mxx`) from [`Uprava/ARTICLES.md`](https://github.com/gasyoun/Uprava/blob/main/ARTICLES.md), and the mode — `deck` (default) or `poster`.

## Phase 0 — Precondition: the paper is accepted or near-final

Presentation material is wasted on a paper whose substance can still move. Fire this only when the paper is **accepted** (a talk/poster slot exists) or **near-final** (readiness 4–5, camera-ready or in-press). If the paper is still under review or below readiness 4 — its figures and numbers are not yet frozen — **stop** and route back to [`/paper-referee`](https://github.com/gasyoun/claude-config/blob/main/commands/paper-referee.md) or [`/paper-submission-pack`](https://github.com/gasyoun/claude-config/blob/main/commands/paper-submission-pack.md). Building slides off a draft that a referee then reshapes is throwaway work.

## Phase 1 — Resolve the paper and mine its substance

Look it up in [`Uprava/ARTICLES.md`](https://github.com/gasyoun/Uprava/blob/main/ARTICLES.md): repo, manuscript path, current readiness, the venue (and its talk/poster format — a lightning talk, a 20-minute slot, and an A0 poster each want a different shape). Read the FULL manuscript and pull:

- **The thesis** — the one-sentence claim the whole paper defends.
- **The motivation** — why it matters, in the terms the venue's audience cares about.
- **The method** — the minimal chain a listener needs to trust the result, not the full paper's detail.
- **The results** — the headline numbers and the figures/tables that carry them.
- **The takeaway** — what the audience should remember and where to find the paper (full blob URL, DOI if minted).

## Phase 2 — Lay out the deck or the poster

### `deck` — a slide outline

Write `<Axx>_talk_outline.md` next to the manuscript (dated header + byline footer per the doc contract), one section per slide, sized to the venue's slot:

1. **Title** — paper title, byline block from [`Uprava/AUTHOR.md`](https://github.com/gasyoun/Uprava/blob/main/AUTHOR.md) (never invent it), venue + date.
2. **Motivation / the problem** — 1–2 slides; the hook line left as an author-voice stub.
3. **Method** — 1–3 slides; the trust-chain, not the full apparatus.
4. **Results** — the load-bearing slides, each built around one **already-committed figure/table** (Phase 3).
5. **Takeaway** — the remember-this line (stub) + paper URL/DOI + contact.
6. **(optional) Backup** — the extra figures a sharp Q&A will demand.

Per slide: a title, the 1–3 bullet skeleton, the figure it shows (path), and a `> SPEAKER NOTE:` line for the spoken framing where register matters (left as a stub).

### `poster` — a section layout

Write `<Axx>_poster_layout.md` next to the manuscript. Lay out the standard poster grid — **Title/authors banner · Motivation · Method · Results (the committed figures, largest) · Takeaway · References/QR-to-paper** — with a size/placement note per block (results dominate; text is minimal; the figures do the work). Byline from [`Uprava/AUTHOR.md`](https://github.com/gasyoun/Uprava/blob/main/AUTHOR.md).

## Phase 3 — Reuse the paper's committed figures — never regenerate

Point every results slide/panel at the paper's **existing committed figure files** by their in-repo path (full blob URLs in the outline). Do **not** regenerate a figure, re-run an estimator, or redraw a chart — the paper's figures are already refereed and camera-ready; a fresh render risks a number that disagrees with the frozen paper. If a figure the talk wants does not exist as a committed artifact, that is a note for the human ("no committed figure for X — author to supply or cut"), not a cue to build one.

## Phase 4 — Leave author-voice content as stubs

The deck's spoken register is the author's public voice and is **not** auto-written. Leave clearly-marked fill-in stubs — `[[HOOK: one-sentence opener — author]]`, `[[TAKEAWAY: the line they remember]]`, `[[TRANSITION]]` — wherever the wording carries the presenter's voice. Scaffold structure, figures, and factual bullets; stub the rhetoric.

## Phase 5 — Bookkeeping

- Deliver the outline/layout file as a branch + PR in the owning repo, auto-merge per the large-work rule (never commit to the default branch from an unattended run).
- This is presentation material, not a readiness change — do **not** bump the paper's readiness in [`Uprava/ARTICLES.md`](https://github.com/gasyoun/Uprava/blob/main/ARTICLES.md) (a talk deck doesn't change how done the *paper* is). Note the deck/poster exists in the paper's row/notes per the [`/articles-update`](https://github.com/gasyoun/claude-config/blob/main/commands/articles-update.md) discipline if a notes field is the convention there, then regenerate the dashboard: `python Uprava/tools/build_dashboard_data.py`.
- The finalize-and-rehearse work is a human action: add an `MG @DO` row to [`Uprava/GTD_NEXT_ACTIONS.md`](https://github.com/gasyoun/Uprava/blob/main/GTD_NEXT_ACTIONS.md) — "fill author-voice stubs + build the actual slides/poster for `<Axx>` and rehearse before `<venue date>`" — linking the outline PR.
- Commit both repos.

Report: the outline/layout PR link, the slide/section count, the list of committed figures reused, the author-voice stubs left to fill, and the GTD @DO.

## When NOT to use

- **A paper not yet accepted** (or below readiness 4) — its figures/numbers can still move → [`/paper-referee`](https://github.com/gasyoun/claude-config/blob/main/commands/paper-referee.md) / [`/paper-submission-pack`](https://github.com/gasyoun/claude-config/blob/main/commands/paper-submission-pack.md) first.
- **Rendering a real slide binary** (PPTX/Beamer PDF) or a print-ready poster PDF — this skill produces the *outline/layout*, not the compiled artifact; building the actual file is the human @DO from Phase 5.
- **The camera-ready paper files themselves** → [`/paper-camera-ready`](https://github.com/gasyoun/claude-config/blob/main/commands/paper-camera-ready.md).

## Safety notes

- **Never regenerate a figure.** Reuse the paper's committed, refereed figures verbatim; a fresh render can silently disagree with the frozen paper's numbers. A missing figure is a human note, not a rebuild cue.
- **Never invent the byline or a DOI.** Author block from [`Uprava/AUTHOR.md`](https://github.com/gasyoun/Uprava/blob/main/AUTHOR.md); an unminted DOI stays a labelled placeholder, not a made-up string.
- **Substance is frozen.** The talk restates the paper's claims — it does not add a result, soften a caveat, or overclaim beyond what the paper defends.
- **Register-sensitive** — the spoken/poster voice is the author's public register; leave it as stubs and run at the Fable 5 (`claude-fable-5`) tier, Opus 4.8 (`claude-opus-4-8`) fallback.

_Dr. Mārcis Gasūns_
