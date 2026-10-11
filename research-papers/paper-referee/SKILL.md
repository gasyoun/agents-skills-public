---
name: "paper-referee"
description: 'Hostile referee-style pre-submission review of one Uprava/ARTICLES.md paper — Major/Minor findings, every figure+claim re-verified, fixes via PR, readiness bumped. Fable 5 (claude-fable-5), Opus 4.8 (claude-opus-4-8). Trigger: "review Axx". Do not use for a friendly authorial rewrite — that is /paper-author-pass; this is the hostile pre-submission review.'

---

# paper-referee

Use this skill when the user asks to run the migrated source command `paper-referee`.

## Command Template


# /paper-referee

_Created: 03-07-2026 · Last updated: 05-09-2026_

The most-repeated unscripted session shape (A16, A36, A13, A19, A21 all ran it by hand in one week): adopt the mindset of a referee looking for reasons to REJECT, verify the manuscript against its own committed data, and apply what survives. `/paper-submission-pack` packages a paper that is already 4–5; this skill is the review that gets it there.

Argument: the stable paper ID (`Axx` / `Mxx`) from `Uprava/ARTICLES.md`, or enough title to resolve one.

## Phase 1 — Resolve and read

Look the paper up in `Uprava/ARTICLES.md`: repo, manuscript path, current readiness, target venue, blocker. Read the FULL manuscript — a referee who skimmed is worthless. Also locate the data the paper is built on (the CSVs/JSON/SQLite it cites or should cite) inside the repo; if the paper's numbers have no committed source, that is itself a Major finding.

## Phase 2 — Hostile review

Review as a domain referee whose default verdict is reject. Hunt in these directions (each is a past real finding, not a hypothetical):

- **Figures vs data** — recompute EVERY table figure and in-text number from the committed source files (A36: all table figures re-verified vs the 3 CSVs; A01/P1: 3 stale figures found). Fan out the `fact-check-against-source` agent for this — one agent per claim family, in parallel.
- **Source claims vs csl-orig** — statements about what a dictionary contains get checked against the actual source text (A36: Bopp/MW72 corrections found this way).
- **Overclaiming / unsourced additions** — anything asserted without a committed artifact or citation behind it.
- **Comparanda and literature gaps** — the obvious analog the venue's referees will name (A36 gained Liddell–Scott/Cambridge Greek Lexicon this way; A01 gained the foundational-metrics lit review).
- **Structure, title, framing** — does the title match what the paper actually shows (A36: history-first retitle)?
- **Methods reproducibility** — could a reader rerun it from what is in the repo? Missing script/data pointers are findings.
- **Ablation / isolation completeness** (method / detector / pipeline papers) — if the claim is "component C improves metric M", demand a remove-one or swap-one row with a committed artifact. A missing ablation table when the scaffold's ablation plan exists (or when the prose implies modular gains) is **Major**. Pure philology papers: `n/a`.
- **Venue fit** — quick check against the target venue's scope; a wrong-field venue is a HOLD finding, not a text fix (A21: Scrinium out of scope).
- **Responsible-NLP / reproducibility checklist gaps** (adopted 11-07-2026 via [H666](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H666-Sonnet_claude-config_acl-paper-checklists_11.07.26.md) — Sonnet 5 (`claude-sonnet-5`), ACL paper checklists) — check
  against the current [ARR Responsible NLP Research checklist](https://aclrollingreview.org/responsibleNLPresearch/)
  (cite the version/date fetched; last confirmed 11-07-2026, "Updated for ARR October
  2024 cycle"): no Limitations section is a **Major** by itself; undocumented data
  provenance/licensing for a used artifact (dataset, dictionary export, code) is Major;
  unreproducible numbers (no committed script/data pointer, no version numbers on
  tools/models) is Major. Missing annotator/compute-budget detail on an otherwise
  reproducible paper, or a thin risk/limitations paragraph, is Minor. Papers with no
  human-subjects component mark the D-family `n/a` and move on — that is not a finding.

Classify every finding **Major** (blocks submission) or **Minor** (should fix), numbered.

**Venue calibration.** This org's default venues (lexicography/DH — see
[ARTICLES.md](https://github.com/gasyoun/Uprava/blob/main/ARTICLES.md)) don't mandate the
formal ARR checklist artifact, but the practices it encodes (state limitations, document
provenance, report reproducibility) are the internal quality bar on every paper reviewed
here regardless of target venue. Only ACL/EMNLP/NAACL/LREC-family targets need the
checklist attached as a formal submission artifact (see `/paper-submission-pack` Phase
3.5) — for other venues, apply the same scrutiny but don't demand the artifact itself.

## Phase 3 — Review memo

Write `<Axx>_review_<tier>.md` next to the manuscript (e.g. `A36_review_fable5.md` — follow the existing naming in that repo), with dated header + byline per the doc contract, full blob URLs on every file reference. Per finding: what/where (section + quote), why a referee would object, the concrete fix, and after Phase 4 the applied-or-not status. State the reviewing model as tier + exact version.

## Phase 4 — Apply fixes

Apply every finding whose fix is text-level and unambiguous, in the same session (the A36/A16 pattern — a memo without the fixes applied is half the value). Findings that need an MG ruling (venue change, scope cut, new experiment) stay open in the memo and go to GTD as @DECIDE instead. Deliver as a branch + PR with auto-merge per the large-work rule; never commit to a repo's default branch from an unattended run.

## Phase 5 — Bookkeeping

- Bump readiness/status in `Uprava/ARTICLES.md` per the `/articles-update` discipline and regenerate the dashboard. A paper that survived a hostile pass with all Majors applied is normally 4/5 (or 5/5 if the venue checklist is also clean — say which and why).
- Mirror any human gate (venue @DECIDE, book-check, send action) to `Uprava/GTD_NEXT_ACTIONS.md`.
- Report: verdict (PROCEED / fixable / HOLD + reason), Major/Minor counts, memo + PR links, new readiness.

## Output

Deliver, every run:
1. **The review memo** — `<Axx>_review_<tier>.md` next to the manuscript: numbered **Major**/**Minor** findings, each with section + quote · why a referee objects · the concrete fix · applied-or-not status · reviewing model tier + exact version, shipped via branch + PR.
2. **The applied fixes** — every unambiguous text-level fix committed the same session; ruling-required items (venue change, scope cut, new experiment) left open in the memo and mirrored to GTD as `@DECIDE`.
3. **Bookkeeping** — `Uprava/ARTICLES.md` readiness/status bumped per `/articles-update` + dashboard regenerated.
4. **A one-line report to the user** — verdict (PROCEED / fixable / HOLD + reason), Major/Minor counts, memo + PR links, new readiness. A memo without the fixes applied is half the value; findings that live only in chat are not delivered.

_Dr. Mārcis Gasūns_
