---
name: "zaliznyak-council"
description: 'Run one, several, or all 9 Zalizniak-family RuWritingStyles style passports as a council pass over a prose document; passport selection is an explicit argument. Triggers: "zalizniak council", "run the Zalizniak passports", "re-council with the other Zalizniak personas", or naming one of the nine passports directly. Mechanical orchestration - any tier.'

---

# zaliznyak-council

Use this skill when the user asks to run the migrated source command `zaliznyak-council`.

## Command Template


# /zaliznyak-council <file-or-dir> [passport-selector]

_Created: 09-07-2026 · Last updated: 05-09-2026_

Wraps the RuWritingStyles `prepare → review → council` pipeline for the nine Zalizniak-family
style passports, so a session can target exactly the passport(s) it needs instead of
improvising CLI flags each time (as [H275 (Fable 5) — SanskritGrammar gasuns-dhatu RWS style passport](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H275-Fable_SanskritGrammar_gasuns-dhatu-rws-style-pass_07.07.26.md)
and its [H415 (Sonnet 5) — gasuns-dhatu RWS edits re-council](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H415-Sonnet_SanskritGrammar_gasuns-dhatu-rws-edits-execute-recouncil_09.07.26.md) follow-up both did by hand). All nine passports are one author's stylistic
personas (A. A. Zalizniak) at different registers/topics — `zalizniak-method` is the
general-purpose one already baked into the named `sanskrit` council; the other eight are
topic-locked (Old Novgorod birchbark letters, nominal declension, enclitics, accentology,
etc.) and will legitimately return **zero findings** on prose outside their domain — that is
a correct null result, not a tooling failure (see H415: 3 of 8 returned nothing on
Sanskrit-topic prose).

Argument 1 — `file-or-dir`: the document(s) to review (same input `prepare` normally takes).

Argument 2 — `passport-selector`, one of:
- a single passport ID (e.g. `zalizniak-udarenie`) — runs just that one;
- a comma-separated list (e.g. `zalizniak-slovo,zalizniak-zametki`) — runs exactly those;
- `all-zalizniak` — runs all 9 (the full family, including `zalizniak-method`);
- `all-zalizniak-minus-method` — the other 8, for topping up a document that already went
  through the named `sanskrit`/`general` council (which already includes `zalizniak-method`) —
  this was H415's actual case;
- omitted — prompt for one of the above rather than silently defaulting to all 9 (an
  accidental full run burns real API budget across N fragments × 9 passports).

## Phase 0 — Resolve passport IDs, don't guess spelling

Run `python3 -m ruwritingstyles.cli list-styles` (from `RuWritingStyles/`) and match against
its output, not against this doc or a handoff's prose — passport-file spelling has drifted
before (`zaliznyak-*` vs `zalizniak-*` inconsistently used pre-H415-rename; if a rename has
landed since this skill was written, `list-styles` is still the source of truth). Resolve the
selector argument to the exact canonical IDs before proceeding.

## Phase 1 — Reachability check (before committing to a big run)

`python3 -m ruwritingstyles.cli provider-status` — confirm the configured backend (DeepSeek by
default per project convention) is reachable. If not, STOP and report what's blocked; never
fabricate review output.

## Phase 2 — Prepare (skip if already segmented)

If the target isn't already split into `runs/<slug>-NN-*` fragments from a prior pass, run
`prepare` first. If fragments already exist (e.g. reusing a prior council's segmentation), do
NOT re-prepare — reuse them, writing this pass's results to a **separate** run-directory
suffix (e.g. `-zalzN`) so you never overwrite another pass's `review.json`/`council.json`.

## Phase 3 — Review + council, driven to actual completion

For each fragment × each resolved passport ID: run `review`, then `council` (or the CLI's
equivalent aggregation step). **Verify completion by content, not by file existence** — a
council step can silently fall back to a truncated/stub result (`status != "completed"`,
zero decisions) on a malformed provider response (seen in H415); check the actual `status`
field before counting a fragment done. If driving this from a background agent: do NOT
hand off to a Monitor-and-end-turn pattern across agent-turn boundaries — background
watches have been observed not to survive an agent's own turn ending (H415, twice). Either
block on completion within one turn's tool calls, or make each resume explicitly re-check
real on-disk state before deciding what's left to do.

## Phase 4 — Curate, don't dump raw findings

Aggregate per-passport results; dedupe against any prior pass's findings by rough
location/content match (exact dedup isn't required — avoid presenting an obvious re-flag as
new). If a passport returned zero or near-zero findings because it's topic-locked away from
the document's subject matter, say so plainly in one line rather than padding the report.
Write a report next to the document's existing style-pass report (same conventions: dated
header, byline, full blob URLs, plain Markdown) — do not silently merge into the original
report file; a follow-up addendum stays a separate, clearly-dated file.

## Hard rules

- Never touch the shared named councils (`sanskrit`, `general`, etc.) to shoehorn in an
  ad-hoc passport list — pass an explicit style list to `review`/`council` if the CLI
  supports it; if it doesn't, that's a CLI gap to report, not a reason to mutate shared
  council config for a one-off run.
- Passport IDs are load-bearing filenames/CLI arguments — if a rename sweep (e.g.
  Zaliznyak→Zalizniak spelling) is pending, do not run this skill against files mid-rename;
  confirm current canonical IDs via `list-styles` first (Phase 0).

_Dr. Mārcis Gasūns_
