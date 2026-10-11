---
name: "paper-author-pass"
description: 'Non-hostile author-voice pass on a near-ready Uprava/ARTICLES.md paper — rewrite via PR, commit SIGNOFF_Axx_author_pass.md, GTD @DO sign row. Fable 5 (claude-fable-5), Opus 4.8 (claude-opus-4-8). Sibling /paper-referee. Trigger: "author pass Axx". Do not use for adversarial pre-submission review — that is /paper-referee; this is the non-hostile authorial-voice pass.'

---

# paper-author-pass

Use this skill when the user asks to run the migrated source command `paper-author-pass`.

## Command Template


# /paper-author-pass <Axx>

_Created: 09-07-2026 · Last updated: 05-09-2026_

Distills the author-voice discipline from [H368](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H368-Fable_Uprava_author_pass_sim_protocol_batch1_08.07.26.md) (Fable — author-pass sim protocol, batch 1; A01 → [csl-atlas PR #228](https://github.com/sanskrit-lexicon/csl-atlas/pull/228), A06 → [csl-atlas PR #229](https://github.com/sanskrit-lexicon/csl-atlas/pull/229)) and the A36 S9 pass ([SanskritLexicography PR #74](https://github.com/gasyoun/SanskritLexicography/pull/74)) — all run by Fable 5 (`claude-fable-5`), improvised each time until this skill. Where `/paper-referee` hunts reasons to REJECT, this pass assumes the substance survived review and makes the manuscript read as the author's own: register, framing, first-person scholarly voice, venue conventions.

Argument: the stable paper ID (`Axx`/`Mxx`) from [`Uprava/ARTICLES.md`](https://github.com/gasyoun/Uprava/blob/main/ARTICLES.md), or enough title to resolve one.

## Phase 1 — Resolve, read, precondition

Look the paper up in `Uprava/ARTICLES.md`: repo, manuscript path, readiness, venue, blocker. Precondition: the paper should be post-referee or otherwise substantively settled (readiness 3–4) — if figures/claims were never verified against committed data, stop and run `/paper-referee` first; a voice pass over unverified substance polishes errors. Read the FULL manuscript plus any existing review memo (`<Axx>_review_*.md`).

## Phase 2 — Author-voice rewrite

Rewrite for voice, not substance. In scope: register and idiom (the author's, per [[user_identity_byline]] — no "Dr." in paper bylines, ORCID/affiliation per the identity block), venue conventions (person, hedging norms, section idiom), framing and transitions, title/abstract alignment with what the paper actually shows, de-agentification (no LLM-flavored phrasing, no filler intensifiers). OUT of scope: numbers, claims, data, citations — any substantive doubt found mid-pass is a finding for the referee lane, parked in the signoff doc, never silently "fixed".

Deliver as branch + PR in the owning repo, auto-merge per the large-work rule.

## Phase 3 — Signoff document (the load-bearing artifact)

Commit `SIGNOFF_<Axx>_author_pass.md` next to the manuscript (dated header, byline, full blob URLs), holding ONLY what a human must still rule on before the bump to 5/5: voice calls made that the author may veto (each with location + one-line rationale), standing verify-flags carried over unresolved, and the explicit read-and-sign instruction. The H368 (Fable, author-pass sim protocol) pattern: the signoff doc is what turns "an agent rewrote my paper" into a 30-minute human review instead of a full reread.

## Phase 4 — Bookkeeping

- GTD: add the `MG @DO · ~30min · Read-and-sign <Axx>` row to [`Uprava/GTD_NEXT_ACTIONS.md`](https://github.com/gasyoun/Uprava/blob/main/GTD_NEXT_ACTIONS.md), linking the signoff doc and the PR (the H368 GTD rows are the template).
- `Uprava/ARTICLES.md` via the `/articles-update` discipline: note the pass, PROPOSE the readiness bump (the bump to 5/5 itself waits for the human sign-off — say so in the row).
- Report: PR link, signoff-doc link, voice calls count, residual flags, proposed readiness. Model attribution tier + exact version everywhere.

## Hard rules

- Never change a number, claim, or citation under the name of "voice" — substance drift in a voice pass is the worst failure mode because nobody re-verifies after it.
- No signoff doc → the pass is not done. A voice rewrite without the residual-calls record forces a full human reread and destroys the skill's value.
- Papers keep the academic byline block (no "Dr."), distinct from the file-authorship byline.

_Dr. Mārcis Gasūns_
