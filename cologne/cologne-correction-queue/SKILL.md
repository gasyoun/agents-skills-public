---
name: "cologne-correction-queue"
description: Validate a csl-orig dictionary correction end-to-end (updateByLine → XML validate → BOM check) and file it into the local monthly batch queue WITHOUT touching csl-orig — the only sanctioned way between monthly batch PRs. Do not use to ship the accumulated queue as a PR — that is /cologne-batch-pr; this only validates and files ONE correction.
---

# cologne-correction-queue

Use this skill when the user asks to run the migrated source command `cologne-correction-queue`.

## Command Template


# /cologne-correction-queue

_Created: 03-07-2026 · Last updated: 06-09-2026_

Argument: `DICT <changefile-path | ISSUE_NUMBER>` (e.g. `mw 1537` or `ap90 C:\...\change_ap90.txt`).

csl-orig is **read-only for direct pushes**; corrections ship as ONE consolidated PR ~monthly (see `/cologne-batch-pr`). This skill does everything up to that point: prove the correction is valid, then park it in the queue. csl-orig is never written to by this skill — validation runs against an isolated tree, so csl-orig's working tree is untouched throughout.

Queue location (git-tracked, pushed same-pass — the [`H2086`](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H2086-Grok_csl-corrections_batch-pending-queue-durability_01.08.26.md) (Grok 4.5) csl-corrections batch-pending-queue-durability contract):
`~/Documents/GitHub/csl-corrections/batch_pending/dictionaries/<dict>/`

---

## Phase 1 — Obtain the change file

- If given a changefile path: use it.
- If given an issue number: fetch it (`gh issue view N --repo sanskrit-lexicon/csl-orig --json title,body,comments`), extract exact old→new text and locate the exact lines in `csl-orig/v02/<dict>/<dict>.txt` (`Select-String` by `<L>` number or content). **Stop and report if ambiguous** — never guess a correction; post no comment (maintainers dislike bot noise) unless the user asks.
- Check it isn't already queued (`batch_pending/dictionaries/<dict>/`) or already recorded as done/rejected in csl-corrections CFR/batch history.

Change file format — paired lines, `;` for comments:
```
NNNNN old [exact full original line]
NNNNN new [exact full replacement line]
```
(`ins` inserts after, `del` deletes.) Write with `encoding='utf-8'`, never `utf-8-sig`.

## Phase 2 — Validate against the delivery base (never mutate csl-orig)

The base is **`origin/main`**, never the local working tree — a stale or feature-branch
checkout silently validates against the wrong line numbers (see `/cologne-batch-pr` Phase 2
step 1 for the measured failure: 20,618/21,817 `mw` records failed once upstream had moved).

Windows (PowerShell):

```powershell
# 0. Make origin/main current before reading anything from it
git -C ~/Documents/GitHub\csl-orig fetch origin --quiet

# 1. Snapshot from the delivery base, not the working tree
git -C ~/Documents/GitHub\csl-orig show origin/main:v02/<dict>/<dict>.txt > <scratchpad>\temp_<dict>_0.txt
$BASE_SHA = git -C ~/Documents/GitHub\csl-orig rev-parse origin/main:v02/<dict>/<dict>.txt

# 2. Apply to the snapshot
python ~/Documents/GitHub\csl-pywork\v02\makotemplates\pywork\updateByLine.py <scratchpad>\temp_<dict>_0.txt <changefile> <scratchpad>\temp_<dict>_1.txt
```

macOS / Linux (sh):

```sh
# 0. Make origin/main current before reading anything from it
git -C ~/Documents/GitHub/csl-orig fetch origin --quiet

# 1. Snapshot from the delivery base, not the working tree
git -C ~/Documents/GitHub/csl-orig show origin/main:v02/<dict>/<dict>.txt > <scratchpad>/temp_<dict>_0.txt
BASE_SHA=$(git -C ~/Documents/GitHub/csl-orig rev-parse origin/main:v02/<dict>/<dict>.txt)

# 2. Apply to the snapshot
python3 ~/Documents/GitHub/csl-pywork/v02/makotemplates/pywork/updateByLine.py <scratchpad>/temp_<dict>_0.txt <changefile> <scratchpad>/temp_<dict>_1.txt
```

Record `$BASE_SHA` — it goes into the queue `readme.txt` line in Phase 3 so the drain can see
how far the base has moved since queue time. (Note: `updateByLine.py` opens files without
`newline=''` only when it runs on Windows — on macOS/Linux the CR-doubling trap below does
not trigger, but the byte-shape assertion stays the cheap belt-and-braces check on both.)

- **BOM check** on the output: first 3 bytes must not be `efbbbf`.
- **Byte-shape assertion (CR-doubling guard).** `updateByLine.py` opens files without
  `newline=''` on Windows, which doubles every `\r` and silently rewrites *every* line in the
  diff (measured: +273,723 bytes over 273,715 `ap90` lines — +1 byte/line). After it runs:
  - `size(temp_<dict>_1.txt)` should be within `size(temp_<dict>_0.txt) + intended_delta ± (N_edits × 200 + 1000)` bytes, where `N_edits` is the number of change-file records and `intended_delta` is their expected net byte change.
  - If the growth exceeds that band, **stop** — CR-doubling (or another bulk rewrite) has happened. Do not queue the output. If you wrap `updateByLine.py` or post-process its output yourself, open files with `newline=''` to avoid re-introducing this.
  - Confirm the line-level diff count between the input and output snapshot equals the intended edit count — anything more is collateral, not the correction.
- **XML validation — isolated tree, csl-orig is never written to.** Adopt the drain skill's
  recipe (`/cologne-batch-pr` Phase 2 step 5) instead of copying into csl-orig and restoring:
  ```sh
  sh generate_orig.sh <dict> tempparent/<dict>
  cp <scratchpad>/temp_<dict>_1.txt tempparent/<dict>/orig/<dict>.txt
  sh generate_pywork.sh <dict> tempparent/<dict>
  (cd tempparent/<dict>/pywork && sh redo_hw.sh && sh redo_xml.sh)
  ```
  Both scripts live in `csl-pywork/v02/`. Pass = "All records parsed by ET" (Windows:
  xmllint unavailable; **macOS ships `/usr/bin/xmllint`** — an extra cross-check is
  `xmllint --noout tempparent/<dict>/pywork/<dict>.xml`, exit 0 = well-formed).
  **Confirm the build consumed your text** — check the staged `orig/`
  file and the generated XML — a green gate on pristine input proves nothing. Remove
  `tempparent/<dict>` when done.
- If `updateByLine.py` errors (line text mismatch): the base has moved since the change file
  was written — re-locate by content against the current `origin/main` snapshot and
  regenerate the change file, or stop and report.

## Phase 3 — File into the queue

Create `batch_pending/dictionaries/<dict>/` if needed and add:

- `change_<dict>_<seq>.txt` — the validated change file (next free `<seq>` in that dir).
- `readme.txt` — append one dated line per entry, prefixed with a per-item status dot (outcome of THIS validation, not a finding's importance): 🔵 clean pass · 🟡 passed with a caveat worth a human glance at batch time (e.g. ambiguous line re-located by content, not by number) · 🔴 does not belong in the queue (validation failed — should have stopped in Phase 2, not been filed). Then: date · issue URL (if any) · one-line description · `validated: All records parsed by ET` · `base: <BASE_SHA short>` (the `origin/main` blob SHA captured in Phase 2, so the drain can see how far the base has moved) · model tier+version that prepared it (e.g. `Fable 5 (claude-fable-5)`).

**Commit and push in the same pass — this is the durable queue, not a local scratch ([`H2086`](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H2086-Grok_csl-corrections_batch-pending-queue-durability_01.08.26.md) (Grok 4.5) batch-pending queue durability; [`H2306`](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H2306-Sonnet_csl-corrections_queue-skill-durability-contract-reconcile_06.08.26.md) (Sonnet 5) queue-skill durability-contract reconcile).** A validated change file that only exists on this machine is a failed delivery, identical in effect to never having validated it:

Windows (PowerShell):

```powershell
cd ~/Documents/GitHub\csl-corrections
git add batch_pending/
git commit -m "corrections: queue <dict> change_<dict>_<seq> for monthly batch"
git push origin HEAD   # or push a feature branch / open a PR if the clone requires one
python scripts/check_batch_pending_tracked.py
```

macOS / Linux (sh):

```sh
cd ~/Documents/GitHub/csl-corrections
git add batch_pending/
git commit -m "corrections: queue <dict> change_<dict>_<seq> for monthly batch"
git push origin HEAD   # or push a feature branch / open a PR if the clone requires one
python3 scripts/check_batch_pending_tracked.py
```

The session does not end until `check_batch_pending_tracked.py` exits **0**. Exit 1 means something in `batch_pending/` is untracked, uncommitted, or unpushed — fix it (add/commit/push) and re-run before stopping, never leave it for a future session to notice.

## Phase 4 — Housekeeping

- Add/refresh one `@WAITING` line in `Uprava/GTD_NEXT_ACTIONS.md` under csl-orig: "N corrections queued for monthly batch PR (run /cologne-batch-pr)". Commit + push Uprava.
- Report: dict, lines changed, validation output line, queue path, and current queue depth across all dicts.

## Change log

Newest first. This section exists so the metadata line at the top can stay two dates long.

| Date | Change |
|---|---|
| 06-09-2026 | H4201 darwin routes |

_Dr. Mārcis Gasūns_
