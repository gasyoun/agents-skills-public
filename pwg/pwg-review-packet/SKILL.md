---
name: "pwg-review-packet"
description: 'Assemble a G5/G6 review packet for promoted-but-unreviewed PWG→RU cards — DE(5-layer)→RU side-by-side cards, HTML sheet via /review-sheet''s contract PUBLISHED to gasyoun.github.io/vote/ (MG ruling 28-08-2026), wire decisions.json → approved via /decisions-apply. Trigger: "G5 sheet for pwg_ru". Do not use for live-readiness or draining a window — that is /pwg-live-gate or /pwg-drain; this only assembles the G5/G6 human review packet.'

---

# pwg-review-packet

Use this skill when the user asks to run the migrated source command `pwg-review-packet`.

## Command Template


# /pwg-review-packet

_Created: 12-07-2026 · Last updated: 05-09-2026_

Argument: what to package for review — a root, several roots, a batch/lane label, or `sample <N>` for a stratified spot-check across the promoted store.

This is the **G5/G6 human gate front-end** for the PWG→RU pipeline — and the pipeline's real **throughput bottleneck**: ~11.5k store rows sit at `review_status='ai_translated'` with G5/G6/G7/G10 human review at 0. `/pwg-drain` → `/pwg-window-close` promote cards to `ai_translated`; those never reach the citable edition (`export_interop.approved_store()` gates on `{approved,human_reviewed}`). This skill packages a batch of them into an interactive voting sheet whose `decisions.json` flips the approved ones to `review_status='approved'`. It is the highest-value work to run **while the generation API is degraded** (it spends no generation tokens).

Working dir: `~/Documents/GitHub/SanskritLexicography/RussianTranslation`. Zero generation tokens — pure selection + rendering + a browser vote.

**Prior art — call the shared emitter, do not reinvent.** The sheet HTML comes from `render_review_sheet(items, config)` in [`csl-pyutil`](https://github.com/sanskrit-lexicon/csl-pyutil) (v0.22.x at last check 26-08-2026 — the emitter contract is versioned and the V1–V8 standard below is stable across it) — the [/review-sheet](https://github.com/gasyoun/claude-config/blob/main/commands/review-sheet.md) standard; **hand-rolled sheet HTML is banned for new sheets**. [`src/build_h180_review_sheets.py`](https://github.com/gasyoun/SanskritLexicography/blob/master/RussianTranslation/src/build_h180_review_sheets.py) predates the emitter: reuse its **selection layer** (the `pwg_ru_translated.jsonl` reader and store filter), but treat its inline HTML template as superseded — port it onto the emitter rather than copying it. The repo's V1–V8 helpers live in [`src/review_sheet_standard.py`](https://github.com/gasyoun/SanskritLexicography/blob/master/RussianTranslation/src/review_sheet_standard.py) (`pwg_entry_href()` supplies each card's V4 `title_href`). Unchanged either way: the `{sheet_id, generated, decided, items:[{id, decision, note}]}` export contract that [/decisions-apply](https://github.com/gasyoun/claude-config/blob/main/commands/decisions-apply.md) consumes. This skill is the **repeatable per-batch generalization**, not a second sheet engine.

**Publish route — MG ruling 28-08-2026: PWG card review sheets PUBLISH on `gasyoun.github.io/vote/` (the [h3361_window25](https://gasyoun.github.io/vote/sheets/h3361_window25_review.html) precedent), never in the SanskritLexicography repo itself.** The sheets embed translated `ru` bodies from the **gitignored/unpublished** store — exactly why they may never be committed to the public SanskritLexicography repo: the generated HTML and per-sheet sample files stay gitignored under `review/` there (only the generator is committed). The sanctioned voting vehicle is the public vote repo [`gasyoun.github.io`](https://github.com/gasyoun/gasyoun.github.io) (branch `master`, root-served): publish at `vote/sheets/<sheet_id>.html` and hand MG the `https://gasyoun.github.io/vote/sheets/<sheet_id>.html` link — never a blob URL. This also satisfies the [28-08 standing policy](https://github.com/gasyoun/Uprava/blob/main/docs/STANDING_POLICY_VOTES_ON_GITHUB_SHEETS_2026.md) that votes happen on published GitHub sheets. Run the [/publish-safety-check](https://github.com/gasyoun/claude-config/blob/main/commands/publish-safety-check.md) sweep before first publication of a new batch — the ruling above is the rights basis for the translated dictionary bodies on this lane; anything outside that scope (personal data, credentials) still stops the publish.

---

## Phase 1 — Select the review-ready cards

From `src\pwg_ru_translated.jsonl`, take rows with `review_status='ai_translated'` (promoted, not yet approved), scoped by the argument. **Exclude synthetic controls and quarantine rows** — any `provenance_class=='synthetic_control'` (the `dq_canary_puregloss` class and the `SYNTHETIC_KEY_RE` keys) and any quarantined/non-`real` provenance never enter a human review packet (they are billed diagnostics, never citable content; the promoter refuses them anyway — C-05). Scope the remaining real rows by:

- a **root / several roots** → all that root's promoted senses;
- a **batch/lane label** → the lane's newly-promoted cards;
- **`judge_sample`** → the audit's own ~10% stratified judge sample for the window, `src\pilot\...\judge_sample.keys.txt` (emitted by `audit_window.py` via `build_judge_sample`) — the cheapest right-sized review scope straight off a just-closed window;
- **`gate-flagged`** → the cards the deterministic gates flagged as residual (the requeue/defect keys) — the reviewer's attention is most valuable here;
- **`sample <N>`** → a deterministic stratified sample across strata/retention bands (reuse `run_batch.py review` worklist / `fidelity_sample.py` / the store filter in `build_h180_review_sheets.py` — **do not rebuild the sampler**).

**Ordering (default: severity-first).** Order the packet by judge/gate severity — worst cards first, so a short review pass catches the most — with corpus stratum (Vedic/Epic/Classical) as a **secondary facet** filter in the same sheet. (This resolves the [H831](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H831-Opus_claude-config_pwg-translation-skills-refresh_12.07.26.md) (Opus 4.8 — PWG translation skills refresh) `@DECIDE` on stratification: severity primary, stratum a facet; if a specific round needs stratum-balance instead — e.g. to keep the print edition's coverage even — a human can flip the primary sort, but severity-first is the standing default.)

Stable ids = the card key (`root~~h0_..`), so a re-generated sheet's votes still line up (`decisions.json` survives regeneration). Skip cards already in an open packet (check `REVIEW_SHEETS_INDEX.md` Awaiting-vote) so a reviewer never votes the same card twice.

## Phase 2 — Render each card for the reviewer

Each card → the current /review-sheet item shape `{ id, filt, title, badges, question, panels, note_placeholder }` (+ `title_href`), with **enough to judge without opening other files**. `context`/`links` are **not** fields — that judgment material goes inside `question` (and `panels` for supporting detail):

- **id** — the card key (`root~~h0_..`); render it visibly per `show_ids` (V3) so a reviewer can cite it back.
- **filt** — the value the filter bar matches: the judge/gate severity bucket (Phase 1's primary ordering), with corpus stratum as the secondary facet.
- **title** — headword + homonym (`aMSa (h0)`), plus the sense count; every card gets a **`title_href`** to the full PWG entry (V4 — build it with `pwg_entry_href()` from [`src/review_sheet_standard.py`](https://github.com/gasyoun/SanskritLexicography/blob/master/RussianTranslation/src/review_sheet_standard.py)).
- **question** — the **DE source side-by-side with the RU translation**: the 5-layer source (`=== LAYER: … ===` banners for PWG main+Nachträge / PW / SCH / PWKVN / NWS) on the left, the promoted RU per-sense rendering on the right, with the **sense tree** (1)/2), a)/b)) and per-sense discrimination visible. Wrap the Russian side with `csl_pyutil.mark_cyrillic()` **after** escaping (V7) — never highlight the German or the IAST.
- **badges** — **provenance** at a glance: the **manifest hash**, **execution route** (`claude-cli-headless`), **profile slot**, **exact model** version; any residual gate flag is its own badge. Keep **sigla verbatim** (ṚV./MBH./m.f.n./`<ls>`/`<ab>`) — they are the reviewer's evidence, never translate them.
- **panels** — the fuller **audit provenance** as `[(heading, html_body), …]` (gate status, requeue rounds, judge verdict), so a reviewer can see exactly which bounded window and profile produced the card without it crowding the judgment view.
- **note_placeholder** — a Russian hint that the note field is where a *partial tweak* is requested instead of an outright reject.
- **Russian labels/prose** (per /review-sheet's default audience), but quoted German/Sanskrit source stays verbatim for citation accuracy.

## Phase 3 — Generate the sheet (delegate to /review-sheet)

Call `render_review_sheet(items, config)` and write its returned string — **never hand-roll the markup or the interactions** (the emitter already carries the tally, keyboard shortcuts, localStorage, File System Access API auto-save + Download fallback, the button-legend footer, and the v0.3.1 dark-theme contrast fix that keeps the note textarea readable in a light-mode browser):

- naming: `sheet_id` = `sanskritlexicography-pwg_ru_<scope>` → `sanskritlexicography-pwg_ru_<scope>_review.html` + `sanskritlexicography-pwg_ru_<scope>_decisions.json` (`<scope>` = the root/batch/`sample-<N>-<date>` that disambiguates repeat rounds — a generic `review.html`/`decisions.json` is banned);
- **mandatory config keys (the 19-07-2026 V1–V8 standard):** `show_ids: True` (V3), `note_min_height_px: 88` (V6), `save_as` bound to `RussianTranslation/review/<sheet_id>_decisions.json` (V8), plus the per-item `title_href` (V4) and `mark_cyrillic()` wrapping (V7) from Phase 2. Add the optional `rating` block (`{"label": "Direct Assessment", "scale": 5, "threshold": 3, "approve_min": 4}`, V1/V5) when the round is a DA-scored fidelity pass rather than a plain accept/reject;
- Russian `title`/`subtitle`/`footer` and Russian `approve_label`/`reject_label`; `filters` from the severity buckets of Phase 1; `generated` passed explicitly as a date string (never computed inside the emitter, so a regenerated sheet stays reproducible);
- three verbs only — ✅ approve · ❌ reject · ⏸ defer; a requested *partial tweak* is a note, never a fourth button;
- `decisions.json` = `{ sheet_id, generated, decided, items:[{id, decision, note}] }`, including unvoted items as `decision:null`.

## Phase 4 — Publish + surface

- Write the HTML + sample under the **gitignored** `review/` folder in SanskritLexicography (never committed there — publish route above); `.gitignore` already covers it — verify, add if missing.
- **Publish the sheet to the vote repo** (MG ruling 28-08-2026): copy `<sheet_id>.html` to `vote/sheets/` on [`gasyoun.github.io`](https://github.com/gasyoun/gasyoun.github.io) (branch `master`; work in a worktree off its default branch — it is a shared deployed repo), commit + push, then verify the live URL renders.
- Hand MG the published link `https://gasyoun.github.io/vote/sheets/<sheet_id>.html` in chat — never a blob URL, never a local `file://` path as the vehicle — with the one-line orientation (N cards, ~minutes, what the pre-marked items mean).
- The voted `decisions.json` (browser download / File System Access save on the published sheet) lands at `RussianTranslation/review/<sheet_id>_decisions.json` — that path stays the `/decisions-apply` contract.
- Add a `@DO` line to [`Uprava/GTD_NEXT_ACTIONS.md`](https://github.com/gasyoun/Uprava/blob/main/GTD_NEXT_ACTIONS.md) linking the **published** sheet URL: "Review N pwg_ru cards in `<sheet>` → save `decisions.json` to `RussianTranslation/review/`" with the consuming command. Commit + push Uprava.
- Register one row in [`Uprava/REVIEW_SHEETS_INDEX.md`](https://github.com/gasyoun/Uprava/blob/main/REVIEW_SHEETS_INDEX.md) under **Awaiting vote** (filename, repo, purpose, item count, **published URL**, date, consuming command) — same pass.
- [`Uprava/tools/review_decisions_watcher.py`](https://github.com/gasyoun/Uprava/blob/main/tools/review_decisions_watcher.py) auto-mints the "apply `<sheet>` decisions" handoff when the voted `decisions.json` lands — no hand-written follow-up @DO needed.

## Phase 5 — Consumer: decisions.json → approved

State the contract explicitly. The voted `decisions.json` is consumed by [/decisions-apply](https://github.com/gasyoun/claude-config/blob/main/commands/decisions-apply.md):

- **approve** → set `review_status='approved'` on the matching store keys → the sense reaches the citable edition via `export_interop.approved_store()`.
- **reject** → logged with the note (the note is where a reviewer requests a *partial tweak* instead of an outright reject — route those to a targeted `/pwg-window-close` requeue, not a blind re-translate).
- **defer** → parked; stays `ai_translated`, re-surfaces next packet.

Preview what a vote set would promote before applying with `python src\export_interop.py --review-status` (dry-run of the bridge). Never hand-flip `review_status` outside `/decisions-apply` — the audit trail is the whole point of the gate.

_Dr. Mārcis Gasūns_
