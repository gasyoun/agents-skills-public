---
name: "headword-export-regen"
description: 'Regenerate a dictionary''s now-2026 unique-key1/key2 headword exports from live csl-orig and diff against the frozen then-2014 snapshot (added/removed/changed delta). Mechanical + BOM-integrity judgment — Sonnet 5 (claude-sonnet-5).'
---

# headword-export-regen

Use this skill when the user asks to run the migrated source command `headword-export-regen`.

## Command Template


# /headword-export-regen

_Created: 14-07-2026 · Last updated: 28-08-2026_


## Default path (seconds — run this first)

```sh
cd ~/Documents/GitHub/SanskritLexicography/HeadwordLists
python headword_diff.py now
```

LLM / judgment residue only after the script report exists (or the script exits non-zero with a real defect). Suspicious `removed` set stays after that report.

The ONE core data-op of the [SanskritLexicography](https://github.com/gasyoun/SanskritLexicography) repo: rebuild a dictionary's sorted-unique headword lists from the **current** csl-orig and measure the delta against the frozen 2014 Cologne snapshot. This is `HeadwordLists/`, the analytical heart of the repo (see the `## HeadwordLists/` section of [SanskritLexicography/CLAUDE.md](https://github.com/gasyoun/SanskritLexicography/blob/master/CLAUDE.md)).

**Do not reinvent the extractor** — the canonical tool already exists: [`HeadwordLists/headword_diff.py`](https://github.com/gasyoun/SanskritLexicography/blob/master/HeadwordLists/headword_diff.py). It carries the validated recipe, the SLP1 collation, the `¦`/`{#..#}` key2 cleanup, and the CODE→csl-orig-dir map. This skill drives it and interprets the result; only fall to a hand-rolled extractor if a dict needs a field/recipe the script doesn't cover.

Argument `$ARGUMENTS`: a DICT code (`MW`, `AP`, `PWG`, `PWK`, `GRA`, `SKD`, `VCP`, `VEI`, …) or `all` (default). The script always processes every list; scope your **reporting** to the requested code.

## Key facts (encode these, don't re-derive)

- **Filename contract:** `{DICT}-unique-{key1|key2}-{N}.txt`, where `N` = entry count = **line count** (the two must always agree; a mismatch is a defect).
- **key1** = normalised computational key (raw `<k1>`, accent-stripped) — use for matching / dedup / joins.
- **key2** = closer-to-printed-source (the print/citation form; retains `-`, `--`/`—`, `/` udātta accent, `(...)`, `*`, `˚`) — use for editorial review, citation, scan-checking. Dedup the unique set on the key, not on entry-body text.
- **BOM is INCONSISTENT across files** — verified on disk: `then-2014/MW-unique-key1-*.txt` **has** a UTF-8 BOM (`EF BB BF`), its key2 sibling does **not** (and uses CRLF); `now-2026/*` are LF, no BOM. Before writing any file by hand: `head -c 3 <file> | xxd`, **preserve the existing BOM state**, never silently add or strip one. All files are UTF-8. (The script reads then-2014 with `utf-8-sig` so it is BOM-tolerant, and writes `now-2026/` as LF/no-BOM by design — that normalisation is the tool's own contract; the preserve-BOM rule is for manual edits to existing files.)
- **Some sources are too large for an editor** — `HeadwordLists/sanhw1.xlsx` (~41 MB), the `*.htm`/`*.html` reference dumps (~16–75 MB), the `PWG/PWK-fehlerhaft-*.txt` error lists. Use streaming/CLI only; never open them with Read.
- **PD** (Deccan) has **no csl-orig source locally** → no now-2026 file; the script skips it. **PWK** is Böhtlingk's *kürzere Fassung* = csl-orig `pw` (not `pwk`).

## Phase 1 — Locate source + baseline

- Confirm the live source: `~/Documents/GitHub/csl-orig/v02/<dir>/<dir>.txt` (map DICT→dir via the script's `CODE2DIR`; e.g. MW→`mw`, PWK→`pw`). Override the root with the `CSL_ORIG_V02` env var if csl-orig lives elsewhere.
- Confirm the frozen baseline exists in [`HeadwordLists/then-2014/`](https://github.com/gasyoun/SanskritLexicography/blob/master/HeadwordLists/then-2014) (`{DICT}-unique-key{1,2}-{N}.txt`). This directory is **read-only** — never regenerate or overwrite it; it is the 2014 reference point.

## Phase 2 — Regenerate the now-2026 lists

Run the canonical writer (it dedupes on the key, SLP1-collates, and names each file with the true new N):

```
cd ~/Documents/GitHub/SanskritLexicography/HeadwordLists
python headword_diff.py now
```

This rewrites [`now-2026/`](https://github.com/gasyoun/SanskritLexicography/blob/master/HeadwordLists/now-2026): key1 = raw `<k1>`; key2 = clean SLP1 print form via `key2_forms()`. It **removes the stale prior file** for each dict+keytype (its N in the filename changed) before writing the new one — so the count in the filename is always current.

## Phase 3 — Recount + verify the filename

For each regenerated file, the filename's `N` must equal its line count. Verify, don't assume:

```
for f in now-2026/${ARG}-unique-key*.txt; do printf '%s  lines=' "$f"; wc -l < "$f"; done
```

A filename-N ≠ line-count is a bug — re-run or fix the name before reporting. (The script derives N from `len(keys)` and writes exactly that many lines, so a mismatch means an interrupted write.)

## Phase 4 — Diff now vs then

Run the default mode for the added/removed/overlap/growth table + full word-level diffs:

```
python headword_diff.py
```

This writes per-list `_diff/<list>.added.txt` / `.removed.txt` and regenerates [`NOW_VS_THEN.md`](https://github.com/gasyoun/SanskritLexicography/blob/master/HeadwordLists/NOW_VS_THEN.md). Read the interpretation guards it already documents:

- **comparable** — same key format both eras → `added`/`removed`/`growth` are genuine headword changes.
- **format-migrated** — the 2014 `<k2>` snapshots are legacy Cologne numeric transliteration (`am2s4a` = aṃśa) while csl-orig is now SLP1; a raw line-diff is ~100 % and **not** a real change. Compare key2 by **set semantics**, never a raw `diff` (order is normalised varṇa-krama collation, not the legacy 2014 order — always `comm`/`sort -u`/the script's set-diff).
- **`removed` = the QA signal.** Headwords present *then*, gone *now* are merges, corrections, or accidental deletions — scan `_diff/<list>.removed.txt` for anything dropped by mistake.

## Phase 5 — Report the delta

For the requested DICT (or all), report: then-N → now-N, `+added / −removed`, overlap %, growth %, and the comparable/format-migrated verdict, citing the exact [`HeadwordLists/`](https://github.com/gasyoun/SanskritLexicography/blob/master/HeadwordLists) paths. Flag any suspicious `removed` set (unexpectedly large, or containing well-known lemmas) as a possible data-loss regression in csl-orig, not just a count change. If [`NOW_VS_THEN.md`](https://github.com/gasyoun/SanskritLexicography/blob/master/HeadwordLists/NOW_VS_THEN.md) changed materially, that is a durable artifact — commit it with the regenerated `now-2026/` files and note the growth deltas in [`CHANGELOG.md`](https://github.com/gasyoun/SanskritLexicography/blob/master/CHANGELOG.md) per the repo's changelog cadence.

## When NOT to use

- Non-headword data — any of the coverage/crosswalk/oracle/union analyses (`heritage_*`, `Catalan-Pujol/`, `build_union.py`, the DCS studies) have their own scripts; this skill is strictly the `*-unique-key{1,2}-N.txt` regen + then/now diff.
- Correcting the dictionary text itself → that is the csl-orig correction workflow: [/cologne-correction-queue](https://github.com/gasyoun/claude-config/blob/main/commands/cologne-correction-queue.md) then [/cologne-batch-pr](https://github.com/gasyoun/claude-config/blob/main/commands/cologne-batch-pr.md). Agents never commit to csl-orig directly.
- Touching `then-2014/` — it is a frozen reference snapshot; regenerate `now-2026/` only.

## Safety

- **Never overwrite `then-2014/`** — read-only baseline.
- **Preserve each existing file's BOM state** on any hand-edit (`head -c 3 | xxd` first); the org "csl-orig files have no BOM" rule does NOT hold in `HeadwordLists/`.
- **Stream the oversized sources** (`sanhw1.xlsx`, `*.htm`, `*-fehlerhaft-*.txt`) — never open with the Read tool.
- csl-orig must be checked out at `../csl-orig/v02` (or point `CSL_ORIG_V02` at it); a missing source makes the script skip that dict rather than emit a wrong/empty list — check the "Skipped" tail of its output.

_Dr. Mārcis Gasūns_
