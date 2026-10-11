---
name: "rws-council"
description: 'Run RuWritingStyles council (real rws run) over a document with a selectable council + optional --journal compliance; forks route to a review sheet. Triggers: "прогони через совет". Not the fixed panels /sanskrit-council, /zaliznyak-council.'

---

# rws-council

Use this skill when the user asks to run the migrated source command `rws-council`.

## Command Template


# /rws-council <file> [council] [journal]

_Created: 11-07-2026 · Last updated: 26-08-2026_

The generic wrapper over the [RuWritingStyles](https://github.com/gasyoun/RuWritingStyles)
review engine (`rws run`): pick a **named council**, optionally a **journal profile**, get
real per-span findings — not a paraphrase. Born in
[H588](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H588-Fable_RuWritingStyles_rws_post_q3_council_skill_a29_11.07.26.md)
as the generalization of the two fixed-panel skills:

- [`/sanskrit-council`](https://github.com/gasyoun/claude-config/blob/main/commands/sanskrit-council.md) —
  the fixed 6-style `sanskrit` panel (minted by [H433](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H433-Sonnet_RuWritingStyles_sanskrit_council_review_skill_09.07.26.md), Sonnet 5 `claude-sonnet-5`); use it when that exact panel is meant.
- [`/zaliznyak-council`](https://github.com/gasyoun/claude-config/blob/main/commands/zaliznyak-council.md) —
  the 9 Zalizniak-family passports, topic-locked.

This skill is for everything else: `general` / `indology` councils, journal-compliance
runs, or ad-hoc panel choice.

## Phase (a) — Preflight (from the repo root)

Repo: `RuWritingStyles` clone at `~/Documents/GitHub/RuWritingStyles` (cd there; the engine reads
`.env` for the DeepSeek key — works from the repo root regardless of where the session
started).

```sh
python3 -m ruwritingstyles.cli councils                 # list available named councils
python3 -m ruwritingstyles.cli provider-status --provider deepseek --strict
```

DeepSeek is the only configured real provider (standing memory:
[[feedback_no_anthropic_key_use_deepseek]]). If the key is missing, STOP and report —
never run a real review on `--provider mock` (mock emits literal placeholder findings,
confidence 0.1; it is for pipeline validation only and must never be presented as a
review result).

## Phase (b) — Run

```sh
python3 -m ruwritingstyles.cli run <doc> --council <general|sanskrit|indology> \
    --execute --provider deepseek [--journal <vya|ppv|vestnik-spbu>] [--run-id <slug>]
```

- `--council` defaults per document subject: Sanskrit-subject prose → `sanskrit`;
  indological but not grammar-centred → `indology`; anything else → `general`. Say which
  you picked and why in one line.
- `--journal` adds the deterministic journal-compliance section (length, citation format,
  IAST scheme, abstract/keywords presence + word limits) to the report — use whenever the
  document targets one of the modelled journals.
- The document must be a file on disk; if the user gave text inline, write it to a temp
  `.md` first and say so.

## Phases (c)–(e) — Inspection, forks, validation

Identical to [`/sanskrit-council`](https://github.com/gasyoun/claude-config/blob/main/commands/sanskrit-council.md)
phases (c)–(e) — follow them verbatim:

1. `python3 -m ruwritingstyles.cli findings runs/<run-id>` (+ `--span <id>` to drill in);
   summarize per disputed span who agreed/disagreed quoting actual findings; unanimous
   spans get one summary line, not detail.
2. Genuine substantive forks → [`/review-sheet`](https://github.com/gasyoun/claude-config/blob/main/commands/review-sheet.md)
   → [`/decisions-apply`](https://github.com/gasyoun/claude-config/blob/main/commands/decisions-apply.md);
   zero forks → say so, skip both.
3. `python3 -m ruwritingstyles.cli validate-run runs/<run-id>` must print
   `OK run artifacts valid` before the pass is done.

If `--journal` was used, also read the «Соответствие журналу» section of
`runs/<run-id>/report.md` and surface every ⚠ item to the user verbatim.

_Dr. Mārcis Gasūns_
