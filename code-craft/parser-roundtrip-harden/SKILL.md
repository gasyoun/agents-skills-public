---
name: "parser-roundtrip-harden"
description: 'Retrofit round-trip tests + fail-loud parsing onto markdown formats tools parse: format module per family, writer-generated fixtures. For silent {} gates, post-regex-fix readers. Design calls Opus 4.8 (claude-opus-4-8); mechanics Sonnet 5 (claude-sonnet-5).'

---

# parser-roundtrip-harden

Use this skill when the user asks to run the migrated source command `parser-roundtrip-harden`.

## Command Template


# /parser-roundtrip-harden

_Created: 17-07-2026 · Last updated: 26-08-2026_

The FINDINGS §82 discipline, made a retrofit recipe. The failure class: a parser reads a markdown file whose format is defined only in prose (a skill telling a *model* how to type it), the two drift, and the parser fails SILENTLY — returns `{}` / matches nothing / matches too much — so the gate it powers becomes a no-op with no signal. Reverse-engineered test fixtures make it worse: they validate the parser against itself, never against the real writer. This hit the same org three times before the class got named (§52 stale hand-typed cache, §54 substring-over-prose, §81 claim-regex `T`-separator mismatch → the concurrency lock was a total no-op), and once as a red test shipped *with* the fix (§82).

Argument: a repo, or a specific parser/tool file. Default: the current repo, full inventory.

Evidence base: Uprava FINDINGS §52 §54 §81 §82 (+ its [H934 (Opus 4.8) — Uprava unify-claim-systems shared module](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H934-Opus_Uprava_unify-claim-systems-shared-module-iso-format_14.07.26.md) / [H938 (Opus 4.8) — exec-claim git-visible ledger merge](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H938-Opus_Uprava_exec-claim-git-visible-ledger-merge_14.07.26.md) updates) §84 §98; canonical worked instance [`Uprava/tools/handoff_claims.py`](https://github.com/gasyoun/Uprava/blob/main/tools/handoff_claims.py) + [`tools/test_next_task_scan.py`](https://github.com/gasyoun/Uprava/blob/main/tools/test_next_task_scan.py) (round-trip + fail-loud + real-file smoke). Dedupe: [`/computed-table`](https://github.com/gasyoun/claude-config/blob/main/commands/computed-table.md) owns derive-don't-store for *values* (counts, tables); this skill owns *format* drift between a writer and a reader; [`/eval-harness`](https://github.com/gasyoun/claude-config/blob/main/commands/eval-harness.md) owns model-output grading, not file parsing.

## Phase 0 — Inventory the parsers

Find every mechanical reader of a human/model-authored file in the target:

- `grep -rn "re.compile\|re.match\|re.search" tools/ src/` filtered to code that reads `.md`/registry/queue/table/config-prose files.
- For EACH parser record five facts: (1) where the format is *defined* (a skill's prose? a module? nowhere?); (2) who *writes* it (a model hand-typing markdown? a CLI? both?); (3) who *reads* it (how many independent regex copies?); (4) what *gate* the parse powers (concurrency lock, eligibility scan, dedupe, sweep); (5) fixture provenance in its tests (hand-authored vs writer-generated vs none).

## Phase 1 — Rank by drift risk

A parser is drift-prone when any of these hold (worst first):

1. **Silent-empty gate.** No-match ⇒ "nothing to do" and the tool proceeds as if healthy (§81: `live_claims() == {}` meant *no locks*, not *parse failure*). These are the ones that hurt.
2. **Format defined only in prose** — a skill instructs a model to hand-type the row; nothing executable owns the format.
3. **Multiple unshared copies** — writer, reader(s), and fixture each carry their own notion of the format (§81 had three).
4. **Hand-authored fixtures** — reverse-engineered from the reader, a tautology (§82's red test).
5. **Width-limited ID regexes** — `\d{3}` style ceilings that silently un-match past H999/1000 (§82 defense 5; reuse the shared `fruit_scan.HID` fragment, `\d{3,}`).
6. **Substring gates over prose** — these are NOT fixed by a better substring; see Phase 5.

## Phase 2 — One shared module per format

For each format being hardened (the `handoff_claims.py` pattern):

- ONE module owns the format: `format_x()` emits, `parse_x()` reads, and every constant (time format, TTLs, column tuple, ID fragment) is co-located so divergence is deliberate and visible.
- Writers become thin CLIs over the module; the skill/prose that told a model to hand-type the row now says "call the CLI" instead. A model can then no longer introduce a format the reader doesn't expect.
- Every scanner/consumer delegates to the module's parser. Three copies → one.
- **Legacy tolerance:** old artifacts in the wild (pre-fix stamps, old sidecar formats) must still parse — add read-only legacy branches (`LEGACY_TIME_FMT`) so the migration window doesn't strand live state.
- If TWO mechanisms serve similar roles on different substrates (git-pushed table vs local O_EXCL sidecar), unify the *plumbing/format*, not necessarily the mechanism — H934 (Opus 4.8) deliberately kept both roles, one format module.

## Phase 3 — Round-trip test + real-file smoke

- `parse(format(x)) == x` as a test per format — drift between writer and reader becomes impossible by construction, not by vigilance.
- **Fixtures: generate with the writer, or capture verbatim from the live file. Never hand-type one.** A hand-typed fixture is the §82 tautology and can ship a red test with a green conscience.
- A smoke test that parses the ACTUAL live file on disk (`test_real_queue_file_parses` pattern) — real-world drift then fails CI the moment it lands, independent of any sample.
- Where a fix's mechanism is subtle, pin it with a **mutation guard**: a test that deletes the load-bearing branch and asserts recovery breaks (the §63 `test_reread_line_is_load_bearing` pattern), so a future edit can't silently remove it.
- Verify each gate **positively fires** on a synthetic hit. A gate that never fires looks identical to a gate that has nothing to catch (§98) — matching zero rows is not evidence of health.

## Phase 4 — Fail loud on suspicious emptiness

- A line that LOOKS like a row (right shape, right leading ID) but won't fully parse ⇒ a named warning on stderr AND in structured output, never a silent skip.
- A parser returning zero results while the file plainly holds candidate rows (`| H### |`-shaped lines present) ⇒ warn. An empty *table* is the healthy common case — don't false-alarm on genuinely empty sections.
- Exclusion/drop reporting is per-row (id + reason + the matched substring), never counts-only — a tally is not a report (§54); an unobservable drop is how 31 of 35 rows vanish for weeks.
- Fail open where hiding is worse than noise: an unresolvable reference leaves the row visible + a warning, never silently withheld (§98, rule 3).

## Phase 5 — Know the class boundary, then record

- **Stale authored data is NOT this class** (§84): there the parser is correct and the *data* lies (a row nobody flipped). Round-tripping can't fix it — route those to close-discipline + periodic sweeps (`/weekly-review`), and note them rather than "hardening" past them.
- **Substring gates over prose fail in both directions at once and the failures mask each other** (§54/§98 — under-match: no writer uses the magic phrase; over-match: a hit blocks unconditionally forever). The fix is structural: extract the reference as an EDGE and resolve it against the registry's actual state, releasing automatically on completion — never a better substring.
- Records, same pass: new gotchas → `/findings-append`; `CHANGELOG.md` entry + `/cut-release`; `.ai_state.md` tick; if the hardening changed a skill's prose contract (Phase 2), update that skill file in the same commit.

## Gotchas

- The §81 regex fix itself shipped with a red test because the fixture matched the OLD regex — hardening a parser without fixing fixture provenance re-arms the trap.
- `mint_handoff.py` never had this class *because* it round-trips — the standard exists in-org; bring readers up to it rather than inventing a new one.
- Don't let the retrofit change live semantics: A/B the hardened scanner over the real corpus and confirm only the intended rows change fate (the §98 fix ended with exactly three rows differing, all explained).

_Dr. Mārcis Gasūns_
