---
name: "census-stat"
description: 'Compute aggregate stats for one DATA_LAYERS_CENSUS.md row with a committed stdlib counting script, write a dated report, run matching repair when defects surface (encoding fix, staleness verdict, supersede call). Triggers: "census stat", "compute stats for X".'

---

# census-stat

Use this skill when the user asks to run the migrated source command `census-stat`.

## Command Template


# /census-stat <repo-or-census-row> [--repair]

_Created: 11-07-2026 · Last updated: 05-09-2026_

Ten Sprint-Lane-B handoffs in one day (H684, H686, H687, H688, H689, H690, H691, H693,
H694, H695 — all Fable 5 (`claude-fable-5`), per-row data-layer census units — see
[FABLE_FOOTPRINT_HOOKS_SKILLS_AUDIT_2026H2.md](https://github.com/gasyoun/claude-config/blob/main/docs/archive/FABLE_FOOTPRINT_HOOKS_SKILLS_AUDIT_2026H2.md)
§3) all re-derived the same five-step shape by hand: pick a census row, write a
one-off counting script, dump a dated report, occasionally repair a defect the
count exposed, then sync the registries. This skill is that shape, encoded once.

Argument: a repo name or a
[DATA_LAYERS_CENSUS.md](https://github.com/gasyoun/Uprava/blob/main/DATA_LAYERS_CENSUS.md)
row description (or infer the target from the current conversation). `--repair`
signals the census is expected to expose a defect worth fixing in the same pass
(encoding, staleness, supersede) — omit it for a pure count.

**Tier:** mechanical + judgment — Sonnet 5 (`claude-sonnet-5`); bump to Fable 5
(`claude-fable-5`) / Opus 4.8 (`claude-opus-4-8`) only when the interpretation
step (is X superseded? is the drift meaningful?) is genuinely open-ended.

## Default path (seconds — run this first, once the register file is named)

```sh
python ~/Documents/GitHub/Uprava/tools/census_stat.py <register-or-census.md>
python ~/Documents/GitHub/Uprava/tools/census_stat.py <register-or-census.md> --slug <slug>
python ~/Documents/GitHub/Uprava/tools/census_stat.py <register-or-census.md> --slug <slug> --json
python ~/Documents/GitHub/Uprava/tools/census_stat.py --selftest
```

Script: [`Uprava/tools/census_stat.py`](https://github.com/gasyoun/Uprava/blob/main/tools/census_stat.py) (H4100, residue rank 5). LLM / judgment residue only after the script report exists: picking the census row / target file (Phase 1) and interpreting a surprising delta stay judgment. Once the file is named, the script counts the buckets (heading sections; table body rows + list items, separator rows excluded), diffs against the previous snapshot persisted under `data/census_stat/<slug>.json` (same-day re-run replaces, distinct dates append), and emits the stats block — bucket table, delta table, total line.

## Phase 1 — Pick the row, check prior art

1. Resolve the target: a `DATA_LAYERS_CENSUS.md §2/§3` row, or a named data
   layer the user pointed at directly.
2. **Reuse before writing a parser** — this is the single most-repeated mistake
   in the ten exemplar handoffs. Check for an existing builder before writing a
   new one:
   - `Uprava/tools/build_dashboard_data.py` already parses `ARTICLES.md`.
   - `Uprava/tools/fruit_scan.py` / `Uprava/tools/next_task_scan.py` already
     parse the handoffs registry.
   - The owning repo's own `tools/`, `scripts/`, or `src/` for a prior
     ingest/count script over the same data.
   Import/extend an existing parser rather than writing a third one
   (MEGABOOK §5.2, `/prior-art`).

## Phase 2 — Compute (committed script, never throwaway shell)

Write a stdlib-only counting script that lands **beside** the report it
produces, in the owning repo (e.g. `tools/<slug>_stats.py` or
`derived-data/<slug>_stats.py`, matching that repo's convention). Never
compute the numbers in an inline shell one-liner or a scratch file — every
number in the report must be reproducible by re-running the committed script.
Compute exactly what the census row asks for: a histogram, a delta table, a
completeness/dedup check, a distribution — no more, no less.

Windows/encoding: `sys.stdout/stderr.reconfigure(encoding='utf-8')` +
`encoding='utf-8'` on any output-capturing `subprocess.run`, no UTF-8 BOM on
write (org convention, hook-enforced on full-file `.py` writes).

## Phase 3 — Write the dated report

A `.md` report in the owning repo: dated header
(`_Created: DD-MM-YYYY · Last updated: DD-MM-YYYY_`), the
`_Dr. Mārcis Gasūns_` byline, model tier + exact version, the script's exact
invocation, and every path/URL as a clickable full blob link. State the
headline numbers up front, then the supporting table(s). If the report has no
natural home (no existing `RESULTS_LOG.md`-style doc), create one dated
section rather than a bare orphan file, and register it per `/findings-append`.

## Phase 4 — Repair (only with `--repair`, or when the count exposes a defect)

The three shapes seen across the exemplars:

- **Encoding repair** (H693 pattern): stream-convert the broken file
  (e.g. cp1251 → UTF-8) to a sibling path, keep the original, recount against
  the fixed copy.
- **Staleness verdict** (H686 pattern): an explicit
  superseded/not-superseded call on an old snapshot — if superseded, recommend
  archive/delete as a GTD `@DO` for a human to execute; **never delete data in
  this skill** — report only.
- **Supersede / dedup call**: flag which of two overlapping assets is
  canonical going forward, with the numbers backing the call.

If the count comes back clean (no defect), skip this phase — that is the
normal, majority case.

## Phase 5 — Sync

1. If a dataset changed or a new one was derived, add/update its row in the
   [kosha manifest](https://github.com/gasyoun/kosha/blob/main/data/manifest/datasets.json).
2. Flip the source `DATA_LAYERS_CENSUS.md` row (counted, with the report link)
   — Uprava takes direct commits to `main`, no PR needed there.
3. For the owning repo's own report/script commit: PR with auto-merge if it's
   a shared/public repo; direct commit if it's a private single-owner repo
   (matches the org default in `/pr-babysit`).
4. `/handoff-close` + the usual hub sync if this run executed a named handoff.

## Report

One line: repo/row censused, headline number(s), repair applied (if any),
commit/PR links.

## When this isn't the right tool

- A one-off ad-hoc question with no committed data layer behind it — just
  answer it, don't spin up a script + report for a single number.
- A full repo capability audit (call graph, silent-failure census) — use
  `/pipeline-audit` instead.
- Deriving a wholly new cross-repo dataset (alignment, crosswalk) rather than
  counting an existing one — that's `/findings-append`'s asset route, not this
  skill.

_Dr. Mārcis Gasūns_
