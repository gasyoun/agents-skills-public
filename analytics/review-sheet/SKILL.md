---
name: "review-sheet"
description: 'Generate an interactive single-file HTML review/voting sheet (approve/reject/defer → download decisions.json) for any batch needing human gating. Use WHENEVER human review of a list is needed — markdown checkbox sheets are banned. Also publishes the sheet to the gasyoun.github.io/vote hub, registers it in REVIEW_SHEETS_INDEX, and runs ≤10-card sheets as INTERACTIVE question-tool rounds (AskUserQuestion/opencode question, ≤4 cards per call, recommendation first — never a link-and-wait row, never prose questions; MG 10-10-2026). Triggers: "review-sheet", "review sheet", "vote sheet", "voting sheet", "лист голосования", "human gating".'

---

# review-sheet

Use this skill when the user asks to run the migrated source command `review-sheet`.

## Command Template


# /review-sheet

_Created: 03-07-2026 · Last updated: 10-10-2026_

Argument: the items to review (a file path — JSON/TSV/MD list — or "from this conversation").

Standing rule: review/gating artifacts are **interactive HTML, never markdown checkboxes**. A checkbox sheet forces M.G. to hand-edit a file; this sheet gives one-click votes and exports a machine-readable `decisions.json` the pipeline can consume.

**Naming rule — no generic filenames, ever.** `review.html`, `decisions.json`, `review_sheet.html`, `pilot_sheet.html` are BANNED as literal filenames. Every sheet and its companion decisions file get a name unique enough to identify on sight once it lands in a flat `Downloads/` folder next to a dozen others from other repos and other rounds:

```
<repo-slug>-<topic>_<scope>_review.html
<repo-slug>-<topic>_<scope>_decisions.json
```


Example: `commentarystrategies-sundarakanda_35-37_review.html` + `commentarystrategies-sundarakanda_35-37_decisions.json` — not `review.html` / `decisions.json`. This is non-negotiable: a generically-named file in `Downloads/` is indistinguishable from every other sheet ever generated (MG rule 05-07-2026, after `review.html` and `renou_pilot_sheet.html` collided with other sheets on disk).

**Language — Russian by default.** Write every sheet's title, per-item labels, context prose, and the legend footer in Russian, unless the user asked for a different language for this specific sheet or the underlying content is untranslatable in place (e.g. quoted source text in Sanskrit/German/English that must stay verbatim for citation accuracy — keep those quotes as-is, translate the surrounding prose). MG rule 12-07-2026 (H779 (Sonnet 5) — nagari2013 okas/guda/sphic attestation verify — follow-up): the default audience for these sheets is Russian-speaking. Ask if genuinely ambiguous (e.g. a sheet whose consumer is an English-only external collaborator).

---

## Phase 0-pre-pre — machine-resolvable NEVER reaches a human (ruling MG, 17-08-2026)

**A vote sheet is for judgment — never for ratifying the consequences of the agent's
own mechanical bug.** Before building ANY sheet, classify every candidate card:

- **evidence-decidable** — the verdict follows from deterministic witnesses (a printed
  source that decides it, an exact-token match, the agent's own flattening/encoding
  bug where "keep as is" is not even a coherent option). These are NEVER rendered as
  cards. The agent applies them directly with a journal + a report (issue comment /
  report doc): the card becomes a row of a REPORT, not a question.
- **judgment** — taste, semantics, priorities, irreconcilably conflicting witnesses.
  Only these go on the sheet.

A sheet that would be mostly evidence-decidable is NOT built — that is a generator
defect, not work for a human. Fill csl-pyutil's screening counts and
`config['non_decision_share']` honestly instead of routing everything to `human`.

## Phase 0-pre — TSV voting is BANNED; personal data is VOTABLE (rules, 16-08-2026)

**A TSV/CSV/spreadsheet column is never a voting vehicle. Ever.** The markdown-checkbox
ban already covered `- [ ]` lists; a "just fill this column" TSV is the same defect wearing
a different extension, and it fails worse — the reviewer gets no legend at the point of
decision, no per-card consequence, no progress state, no export, and no way to tell an

A TSV remains fine as **machine input or output** (a scan's aggregates, a rename plan an
operator executes). The ban is on asking a *human to adjudicate* inside one.

**A personal-data payload never downgrades the vehicle.** If the batch needs human
gating and its content happens to be personal data — names, participant identifiers,
anything under 152-ФЗ — it still gets a real sheet: same emitter, same cards, same
`decisions.json`. Do **not** fall back to a TSV column, a markdown list, or "a human
does this by hand": that is the checkbox ergonomics this skill exists to abolish, and
sensitivity is not a reason to hand someone worse tooling. Specimen: the H2839
`<client>/` path-name gate was first shipped as a 41-row TSV and was unvotable in
practice.


**Screen before you render** (Phase 0-bis, class (b)) — blind lists written *before* the
data is opened take cards off the plate without anyone reading them, and for PD that is
worth more than usual: it is the [FINDINGS §414](https://github.com/gasyoun/Uprava/blob/main/FINDINGS.md)
inversion applied to the gate itself. H2839: 24 clusters → 19 cards via a self/deity/author
list.


## Phase 0 — Is a sheet the right vehicle at all? (gate, MG rule 19-07-2026)

**A one-question decision goes in CHAT, not in a sheet.** Generating a file, opening a
browser, downloading an artifact and running `/decisions-apply` is the right cost for a
*batch*; for a single judgment it is pure overhead on both sides. Ruled by MG after voting on
`review-vedic-r02-adjudication` — a 1-item sheet: «Голосование с одним вопросом должно быть
в чате, а не отдельной HTML страницей».

| Items to judge | Vehicle |
|---|---|
| 1 | ask in chat, in one message, options enumerated |
| 2–4 | chat, one message — still cheaper than a sheet round-trip |
| ≥5, **or** repeated identical judgments (per-correction, per-card, per-headword, per-issue) | generate the sheet |

Borderline (4–6 items) → prefer chat if each item needs different framing; prefer a sheet if
the items are homogeneous enough that one legend covers them all.

**Every option must carry its CONSEQUENCE, not just its label.** This holds in both vehicles,
and it is the second half of the same ruling («не написано, к чему приведет тот или иной
выбор»). An option is not "what this position is called" — it is *what changes, in which file,
and what claim stops holding*:

Write the consequence clause for **each** option before generating anything. If you cannot
state one, you do not yet understand the decision well enough to ask for it.

**Content-grade bar (H2046 (Fable 5) — review-sheet content-grade standard, 31-07-2026) — additive to V1–V8.** Phase 0 + emitter UX can be
green and the card still unanswerable (specimen: H1682 (Sonnet 5) — h1303 abbrev rule collapse — `caus.→кауз.` without expansion,
frequency, case-fold, or dictionary examples). Before generating or remaking a sheet, score
each card type against
[Uprava/docs/REVIEW_SHEET_CONTENT_STANDARD_2026.md](https://github.com/gasyoun/Uprava/blob/main/docs/REVIEW_SHEET_CONTENT_STANDARD_2026.md):
on a non-N/A checklist row **blocks ship** — remake or shrink the ask; do not offer the
sheet for vote. Index pointer: [REVIEW_SHEETS_INDEX.md](https://github.com/gasyoun/Uprava/blob/main/REVIEW_SHEETS_INDEX.md) header.

## Phase 0-bis — Screening gate ([H1649](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1649-Opus_Uprava_voting-machine-screening-gate-orgwide_26.07.26.md) (Opus 5) — voting-machine screening gate org-wide, MG 26-07-2026)

**No card reaches a human that an agent or dataset can already decide.** Between Phase 0
("is a sheet the right vehicle?") and Phase 1 ("normalize the items"), for **every**
candidate card state which of these resolved it and how:

| Class | Resolver | What you record |
|---|---|---|
| **(a) deterministic** | a check over the item's own structure (slot counts, markup skeleton, Cyrillic-in-`{#…#}`, duplicate id already voted elsewhere) | rule name + one-line evidence |
| **(b) dataset lookup** | a dataset that already exists **in this repo or another** (`citation_tm`, ACC×NCC crosswalk, agent verdict TSV, TM, corpus.db) | dataset path + key + hit |
| **(c) agent adjudication** | an agent pass with **cited** evidence (В2 pattern) | verdict + citation + provenance `agent` (never `human`/`gold`) |
| **(d) human required** | nothing above settles it (taste, gold calibration, research label IS the deliverable) | reason why (a)–(c) fail |

**Precision before recall.** A screening rule that manufactures false positives (e.g. flagging
Latin/IAST glosses as "untranslated German" — audit D5) costs more human time than it saves.
Publish per-rule precision for every rule that can false-positive. Probe:
[`Uprava/tools/screen_cards.py`](https://github.com/gasyoun/Uprava/blob/main/tools/screen_cards.py).
Audit: [VOTING_SHEET_SCREENING_AUDIT_26-07-2026.md](https://github.com/gasyoun/Uprava/blob/main/docs/VOTING_SHEET_SCREENING_AUDIT_26-07-2026.md).

**Do not** delete or regenerate a voted sheet to apply screening. Do not drop pending queues
in place without a human ruling — H1649 deliverable 4 is a *proposal* of what screening
would remove.

## Phase 0-ter — Card substance rules (MG ruling 15-08-2026, from the `h1306_style` vote)

Emitter-compliant and screened is not the same as *answerable*. MG voted nine cards and
returned five rules that now bind every sheet — a policy card that breaks them is a defect,
even if V1–V8 and the content-grade bar are green. Evidence:
[H2804](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H2804-Opus_SanskritLexicography_h1306-style-vote-apply_15.08.26.md) (Opus 5) — h1306 style vote apply
· guide §5.4 in [PWG_RU_STYLE_GUIDE_OF_RECORD_2026-07.md](https://github.com/gasyoun/SanskritLexicography/blob/master/RussianTranslation/pwg_ru/PWG_RU_STYLE_GUIDE_OF_RECORD_2026-07.md).

1. **5–10 examples per rule, never one.** A single specimen does not show a reviewer whether
   the rule generalises. Verbatim: «Почему приведен 1 примеров? 5-10 примеров на каждый
   случай, сделай это правилом.»
2. **Every percentage names its denominator, in the card.** «18,5 % односложных DE-глосс» is
   unanswerable until the card says *of what* — the translated subset or the whole set. A
   share quoted against a working store is quoted against the wrong base if the claim is
   about the dictionary: count against the headword list and say so.
3. **Say what approve and what reject will DO.** A card whose Against-block lists authorities
   ("rejected by all three") without stating the consequence of each button leaves the
   reviewer guessing which way the ballot runs. MG could not tell, and deferred two cards for
   exactly this. Rewrite in plain language until both branches are explicit.
4. **Do not repeat the same literature citations 1:1 across every card.** Cite the supporting
   passage once, or vary the quotations and the authors per card. Nine identical citation
   blocks are padding, not support.
5. **Anglicisms in Cyrillic dress are not Russian.** In human-facing card text write
   «словник» / «база карточек», not «стор»; keep Latin `store` for the technical object only.

Corollary for the applier: **a sheet may come back with every option rejected and a sixth
option written into the note.** That is a ruling, not a null result — record it, do not
re-serve the rejected options, and mint the successor sheet for its deployment only.

## Phase 1 — Normalize the items

Parse the input into a JSON array; each item gets the exact shape `render_review_sheet()`
consumes (Phase 2) — do not invent a different shape (`context`/`links` are NOT current
fields; see the H807 note below — H807 (Fable 5) — Kochergina methodichka v1):
`{ "id": stable-id, "filt": the value the filter bar's buttons match against (data-filt) — e.g. a category/status key, "title": one-liner, "badges": [optional short tags shown next to the title], "question": HTML the reviewer judges (the entry text, old→new diff, source quote, links — enough to decide WITHOUT opening other files), "panels": [(heading, html_body), ...] (optional supporting detail, each rendered as its own collapsible-looking block), "note_placeholder": optional placeholder text for the free-text note field }`

Stable IDs matter — decisions must survive regeneration of the sheet. Reuse the item's natural key (issue number, `<L>` number, line number), not array position.

## Phase 2 — Generate the sheet

**Use the shared emitter, don't hand-write HTML.** `render_review_sheet(items, config, *,
extras=True, screening=..., manifest=None)` in [`csl_pyutil`](https://github.com/sanskrit-lexicon/csl-pyutil)
(H925 (Sonnet 5) — review-sheet emitter extract; `screening=` since H1649 / v0.8.0) is the canonical implementation of everything below
— one tested, browser-verified emitter instead of re-deriving the same markup/JS from scratch
on every invocation. The real installed signature takes positional `items` + a `config` dict,
not flat kwargs — a caller hit the drift at H807 and had to introspect the package to rewrite
around it; this doc now matches the installed package (`csl_pyutil.__version__ >= "0.8.0"`
for the screening gate):


### Card-first voting rule — clarity before controls (12-09-2026)

After **Approve**, **Reject**, **Defer**, or keyboard navigation, the next card must land with its **title fully visible below any sticky header, filter, or tally**. Never centre a card on its action buttons and never place its heading underneath a fixed bar. Use `scrollIntoView({ block: 'start' })` only when the layout leaves enough `scroll-margin-top` for the full card title; otherwise set that margin or scroll to a title anchor. Test this at ordinary laptop width in Chrome.

Write every card so a child can say what is being decided after reading only the title and the first two short lines:

```
What is it?  → one short fact
What changes? → one short result
Choose: ✓ do it / ✕ keep it / ⏸ later
```

- Put the decision in the title; do not make the reader infer it from a dense paragraph.
- Use short words, one idea per line, and no unexplained internal terms. Move technical evidence behind a labelled link or a collapsed detail.
- Prefer a tiny diagram when it explains a chain faster than prose: `now → change → result`. Do not add decoration-only graphics.
- Keep the options and their consequences close together; the buttons are secondary to understanding the choice.

## Phase 3 — Place and surface

- Send the file to the user (SendUserFile, render).
- Add a `@DO` line in `Uprava/GTD_NEXT_ACTIONS.md`: "Review N items in <sheet> → save decisions.json to <expected path>" with the consuming command. Commit + push Uprava.
- **`config["personal_data"] is True` sheets skip BOTH the public hub and the inbox** — the same manual exclusion H2839 already applied by hand for a sheet whose content itself was personal data (participant names in a corpus path listing), now a formal config flag instead of a one-off judgment call. Phase 3-bis does not run for these; neither does any inbox/device-flow step. Local-only, exactly like H2839.

## Phase 3-bis — Publish to the public vote hub (DEFAULT, never ask)

**Standing rule, MG 15-08-2026: every voting sheet is published to <https://gasyoun.github.io/vote/> in the same pass that generates it — by default, with no permission question.** A `file:///C:/Users/...` link means the vote can only be cast on one machine by whoever has the repos cloned; that was the whole reason the hub exists (H2755 (Opus 5) — public vote hub for review sheets). "Copies, not a pipeline" was H2755's own residual — this phase is the pipeline.



## Phase 3-ter — Small-sheet vote mode: ≤10 cards vote INTERACTIVELY via the question tool (MG 10-10-2026 «10 карточек — always interactive»; was «chat, card by card», 26-08)

Enforced by claude-config [stop_small_sheet_interactive_check.py](https://github.com/gasyoun/claude-config/blob/main/hooks/stop_small_sheet_interactive_check.py) — a parked ≤10-card vote with no question-tool use in the session blocks the stop (escape `INTERACTIVE-NA` for a human-ruled defer or cards needing the hub sheet's evidence panels).

1. Present the cards **as interactive question-tool rounds** (AskUserQuestion / opencode `question`), ≤4 questions per call, one card per question, each with its evidence summary in the option descriptions and the recommended course marked "(Recommended)". Round caps mean a 10-card sheet = 3 calls (4+4+2) — that is one sitting, not a defect.
2. **MG's interactive answers ARE the official vote.** No `decisions.json` download, no file drop into `Uprava/review/`, and no prose-question fallback.
3. Transcribe the verdicts into a dated `review/weekly/archive/decisions_applied_<date>_<sheet>.md` audit document naming the provenance («voted interactively via question tool»), flip the [Uprava/REVIEW_SHEETS_INDEX.md](https://github.com/gasyoun/Uprava/blob/main/REVIEW_SHEETS_INDEX.md) row to VOTED+APPLIED with that note, close the GTD vote row, and only then execute what was approved. Worked example: the nkrya_showcase_bib_10 vote (10 cards, 3 rounds, 10-10-2026).
4. **Phase 3-bis publish is NOT cancelled** — the hub copy still gets generated and pushed exactly as above; it just stops being the vote *surface* for small sheets and stays archival card text (H2968 (Opus 5) — skill-mine sheets to vote hub, above).

**Boundaries:** large sheets (>~10 cards, e.g. a 36-row GTD triage or a 100-row spot-check) stay on the hub flow — interactive chat would be worse for the human there; export-to-JSON or batch wording remains available on request. A chat vote is still a human-gated decision — no building before the words arrive, and a «нет» on a policy card means "build it" exactly as printed. MG asking for the hub/JSON flow explicitly for any small sheet wins for that sheet.
## Phase 4 — Document the consumer

State in the report exactly what will read `decisions.json` and where to drop it. If nothing consumes it yet, say so — the sheet's contract (schema above) is what the future consumer codes against.

## Phase 5 — Register in the sheet index

Add one row to [`Uprava/REVIEW_SHEETS_INDEX.md`](https://github.com/gasyoun/Uprava/blob/main/REVIEW_SHEETS_INDEX.md) under **Awaiting vote**: sheet filename, repo, purpose (one line — what's being decided and why), item count, the **hosted `gasyoun.github.io/vote/` URL from Phase 3-bis** (plus the local path as the fallback), date generated, consuming command. This is the single place that answers "what sheets are waiting on me" and "what was this sheet even for" months later — do this in the same pass that generates the sheet, not as a follow-up. Commit + push Uprava alongside the GTD row.

_Dr. Mārcis Gasūns_
## References

Detail moved verbatim (read on demand — lives in the rendered skill copy's references/):

- `references/gate-examples-and-hosting.md` — Phase 0 family specimens — machine-resolvable origin, TSV specimen, PD placement table, hosting nuance, consequence examples, worked screening examples + emitter screening code
- `references/content-grade-standard.md` — Content-grade bar detail — U1–U7 / T2 C1–C8 card-type enumeration (H2046)
- `references/emitter-reference.md` — Naming-field definitions; Phase 2 emitter reference — install, render_review_sheet signature/config, V1–V8 standard, contrast requirement, produced-feature list
- `references/phase3-gates.md` — Phase 3 evidence gate + readiness gate bullets (H4114/H4116, H4857) — full gate checklists and standing policy links
- `references/hub-publish-and-packs.md` — Phase 3 detail — repo placement/deletion rule, decisions watcher, packset emitter, hub publish steps 2–5, hub rights
- `references/small-vote-history.md` — Small-vote policy history + voter-fatigue remedies (04-09-2026)
