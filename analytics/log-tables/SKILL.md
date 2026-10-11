---
name: "log-tables"
description: 'Capture substantive result/evidence tables from current chat into a durable file (repo doc or append-only RESULTS_LOG.md) so they survive the session. End of a metrics/census-tables session, or "log the tables".'

---

# log-tables

Use this skill when the user asks to run the migrated source command `log-tables`.

## Command Template


# /log-tables

_Created: 07-07-2026 · Last updated: 05-09-2026_

## Default path (seconds — run this first, once Phase 1 has picked the table + home)

```sh
python ~/Documents/GitHub/Uprava/tools/log_tables.py --input data.json --context "..." --source "..."
python ~/Documents/GitHub/Uprava/tools/log_tables.py --input data.json --context "..." --source "..." \
    --target RESULTS_LOG.md --apply
python ~/Documents/GitHub/Uprava/tools/log_tables.py --input data.json --context "..." --source "..." \
    --target README.md --heading "## Metrics" --apply
python ~/Documents/GitHub/Uprava/tools/log_tables.py --selftest
```

Script: [`Uprava/tools/log_tables.py`](https://github.com/gasyoun/Uprava/blob/main/tools/log_tables.py) (H4101, residue rank 6). LLM / judgment residue only after the script report exists: deciding WHICH tables in the chat are worth keeping (Phase 1) and picking the existing-doc-section vs `RESULTS_LOG.md` home (Phase 2, step 1 vs step 2) stay LLM. Once a table + home are picked, the script renders the deterministic GFM table, builds the dated `## DD-MM-YYYY — <context>` entry with the `_Source: ...`_ footer, and appends it — deduping by date+context so a repeat run is a no-op. Dry-run by default (prints the entry); `--apply` writes.

Tables produced only in a chat evaporate when the chat ends. This command sweeps the current
conversation for the tables worth keeping and writes them into a durable file with provenance, so a
future session (or a human) can recall them. It is the bulk / end-of-session counterpart to the
standing **"persist substantive tables — never chat-only"** convention (which fires inline, per table);
use this when a long session accumulated several tables at once, or the user asks to log them.

Argument (optional): a target hint — a repo, a specific file, or "all". Infer from the conversation if
omitted.

## Phase 1 — harvest

Scan the conversation (this turn's context) for Markdown tables. **Keep** the substantive ones:
- result / metric tables (verdict distributions, counts, rates, yields),
- before/after or A/B comparisons, tier/bucket breakdowns,
- confusion matrices, censuses, frequency distributions,
- any table the user reacted to or asked about.

**Drop** the ephemeral ones: `AskUserQuestion` option lists, a 2-row scratch count used mid-derivation
and then superseded, a table that is already committed verbatim in a repo file (check first — don't
duplicate). When in doubt, keep it (a recallable table is cheap; a lost measurement is not).

## Phase 2 — route each table to its durable home

For each kept table, pick the home in this order:
1. **An existing repo doc with a natural section** — if the table belongs in a README / design doc /
   `.ai_state.md` that already covers the topic, add it there under the right heading (this is usually
   better than a generic log: the table sits next to its explanation). This is what "update the readme"
   means.
2. **A repo `RESULTS_LOG.md`** (create at repo root if absent) — for tables with no natural doc home,
   or a running series of measurements. Append-only, reverse-chronological. Dated header +
   author byline per the org MD contract (the `_Created: DD-MM-YYYY · Last updated: DD-MM-YYYY_`
   + `_Dr. Mārcis Gasūns_` byline form enforced by [`/md-hygiene`](https://github.com/gasyoun/claude-config/blob/main/commands/md-hygiene.md) Phase 2). The default-path script emits both the `RESULTS_LOG.md`-fresh header and the entry in one call (`--target RESULTS_LOG.md --apply`, no `--heading`). Each entry:

   ```markdown
   ## DD-MM-YYYY — <one-line context: what it measures, which command/handoff produced it>

   <the table>

   _Source: <script/command or handoff H###; session model tier + exact version>._
   ```
3. **Scratch only** — if the repo is not the right place (e.g. cross-repo infra measurement), route via
   `/findings-append` to the correct hub instead.

Rules for committed Markdown: full `blob`/`tree` URLs (never relative), plain Markdown (no HTML),
`DD-MM-YYYY` dates, model attribution as **tier + exact version** (resolve aliases). Dedupe: if the
target already holds this table (same caption/data), update in place rather than appending a copy.

## Phase 3 — provenance & commit

Every logged table carries: the date, a one-line context, and what produced it (script, `run_all.py`,
a handoff `H###`, the model tier+version). Then commit per the repo's git discipline — if a handoff
already scopes the work, commit → PR → merge with no confirmation ask (the handoff-scoped rule); for
the private `Uprava` hub, direct push to `main`.

## Guardrails

- Never persist a table already committed elsewhere — link to it instead.
- Never persist secrets, tokens, or unpublished personal data lifted into a table.
- Keep captions honest: if a number is from a superseded run, say so (e.g. "pre-rebuild reprocess") —
  don't present a stale table as current.
- This is additive documentation, not a data release: for a table that IS a citable dataset, use
  `/data-release` instead.

_Dr. Mārcis Gasūns_
