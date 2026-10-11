---
name: "sanskrit-council"
description: 'Run the named 6-style sanskrit RuWritingStyles council over a document (elizarenkova-veda…lidova-commentary), inspect decisions, route genuine forks to human review. Any tier. Triggers: "run the sanskrit council", "6-style council review".'

---

# sanskrit-council

Use this skill when the user asks to run the migrated source command `sanskrit-council`.

## Command Template


# /sanskrit-council <file-or-dir>

_Created: 09-07-2026 · Last updated: 26-08-2026_

Wraps the existing [RuWritingStyles](https://github.com/gasyoun/RuWritingStyles) council
machinery — [council.py](https://github.com/gasyoun/RuWritingStyles/blob/main/src/ruwritingstyles/council.py)
+ [council_summary.py](https://github.com/gasyoun/RuWritingStyles/blob/main/src/ruwritingstyles/council_summary.py)
+ the named `sanskrit` panel in [styles/manifest.yml](https://github.com/gasyoun/RuWritingStyles/blob/main/styles/manifest.yml) —
so a session can run the panel and inspect its decisions without re-deriving the choreography
from CLI flags each time. Confirmed 09-07-2026: the committed `sanskrit` council is exactly
the 6-style panel MG runs in Fable chats — `elizarenkova-veda`, `toporov-etym`,
`panini-traditional`, `zalizniak-method`, `tronsky-readings`, `lidova-commentary`. No delta
from the manifest; nothing to park as `@DECIDE`.

This is a **different** panel from [`/zaliznyak-council`](https://github.com/gasyoun/claude-config/blob/main/commands/zaliznyak-council.md),
which drives the nine Zalizniak-family passports only (topic-locked style/register work, not
Sanskrit-subject review). `zalizniak-method` is the one passport the two panels share. Use
this skill for the standard 6-style Sanskrit-subject review; use `/zaliznyak-council` to
target one or more of the other eight Zalizniak passports specifically.

## Phase (a) — Preflight

From `RuWritingStyles/`, run:

```sh
python3 -m ruwritingstyles.cli provider-status --provider deepseek --strict
```

DeepSeek is the primary real provider for this project — there is no Anthropic key
configured (standing memory: [[feedback_no_anthropic_key_use_deepseek]]). If DeepSeek isn't
ready, STOP and report what's blocked rather than falling back to `--provider mock` for a
real review (mock is for pipeline validation only — every mock finding reads literally
`Mock provider placeholder finding for pipeline validation`, confidence 0.1, and must never
be presented as a real result).

## Phase (b) — Run the council

```sh
python3 -m ruwritingstyles.cli run <doc> --council sanskrit --execute --provider deepseek
```

Optionally pin `--run-id <slug>` for a predictable output directory
(`runs/<run-id>/`); otherwise the CLI assigns one. This single `run` invocation drives
prepare → review (all 6 styles) → council → revise → verify end-to-end — no need to call
`review`/`council` per-style manually.

## Phase (c) — Inspection

Read the aggregated decisions and per-span findings:

```sh
python3 -m ruwritingstyles.cli findings runs/<run-id>
python3 -m ruwritingstyles.cli findings runs/<run-id> --span <span_id>   # drill into one disputed span
```

Also read `runs/<run-id>/council.json` directly for the raw per-style verdicts. Summarize,
per disputed span:
- which styles agreed and on what,
- which styles disagreed and why (quote the actual finding, not a paraphrase),
- any span the council deferred on (no majority verdict).

A span where every style that fired agrees is not a fork — don't pad the summary with those;
name them in one line ("N of M spans unanimous") and spend the detail on the disagreements.

## Phase (d) — Genuine forks → human review

For each span where styles genuinely disagree (not just phrasing — an actual substantive
call the tool can't make), route it through [`/review-sheet`](https://github.com/gasyoun/claude-config/blob/main/commands/review-sheet.md)
(interactive HTML, never a markdown checkbox list — see
[[feedback_interactive_review_not_checkboxes]]). Once decisions are recorded, apply them via
[`/decisions-apply`](https://github.com/gasyoun/claude-config/blob/main/commands/decisions-apply.md).
If there are zero genuine forks, say so and skip both steps — don't manufacture a review
sheet for a report with no disputes.

## Phase (e) — Validate before calling it done

```sh
python3 -m ruwritingstyles.cli validate-run runs/<run-id>
```

Must print `OK run artifacts valid` before the pass is considered complete.

## Verified 09-07-2026 ([H433](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H433-Sonnet_RuWritingStyles_sanskrit_council_review_skill_09.07.26.md) — Sonnet 5, sanskrit council review skill)

A full mock-provider dry run (`--execute --provider mock`) exercised phases (b)–(e)
end-to-end against a real document (`runs/resume-test-1/original.md`) at
`runs/h433-sanskrit-council-test/`: all 6 styles reviewed, `council.json` produced,
`findings` listed one disputed span (`p002`) with all 6 styles' mock placeholder verdicts,
and `validate-run` returned `OK run artifacts valid`. This confirms the choreography works
end-to-end; a real `--provider deepseek` run is still required for substantive (non-mock)
findings.

_Dr. Mārcis Gasūns_
