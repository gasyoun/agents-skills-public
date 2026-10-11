---
name: "cologne-batch-pr"
description: 'Monthly csl-orig delivery — re-validate everything in the local correction queue against upstream, open ONE consolidated pull request. Never more than ~monthly; csl-orig tolerates no PR noise. Triggers: ship the csl-orig queue, monthly csl-orig batch. Do not use to validate or file a single correction — that is /cologne-correction-queue; this only ships the accumulated monthly batch as one PR.'
---

# cologne-batch-pr

Use this skill when the user asks to run the migrated source command `cologne-batch-pr`.

## Command Template


# /cologne-batch-pr

## Default path (seconds — run this first)

The mechanical ship loop (branch → stage → commit → push → `gh pr create`, per repo across a repo list) is ported to a deterministic script (H5489) — dry-run plan by default, `--apply` for the real run:

```sh
python ~/Documents/GitHub/Uprava/tools/cologne_batch_pr.py --help
# dry-run plan (default; no writes, prints per-repo plan):
python ~/Documents/GitHub/Uprava/tools/cologne_batch_pr.py --repo-list repos.txt \
  --branch batch/202610-corrections \
  --message "corrections: 2026-10 batch (N changes across M dicts)" \
  --title "Corrections batch 2026-10" --body-file pr_body.md
# real run (only after the dry-run plan is reviewed):
python ~/Documents/GitHub/Uprava/tools/cologne_batch_pr.py --repo-list repos.txt \
  --branch batch/202610-corrections --message "..." --title "..." \
  --body-file pr_body.md --apply
```

LLM / judgment residue only after the script report exists

What stays the LLM gate (never the script's): the per-repo PR body wording — the per-dict summary table, the `Closes #N` cross-reference list, the workflow provenance paragraph. The script ships `--body-file` verbatim and never authors it; every Phase 2 validation ruling (drop/relocate/regenerate) is likewise made before the script is invoked.

_Created: 03-07-2026 · Last updated: 26-09-2026 (H5489: mechanical route ported)_

Companion to `/cologne-correction-queue`. Drains `csl-corrections/batch_pending/` into a **single** pull request against `sanskrit-lexicon/csl-orig`. One PR per month is the entire noise budget for this repo — never split by dictionary, never comment on individual issues beyond the PR's own cross-references.

Direct pushes to csl-orig's default branch remain forbidden; a topic branch pushed for the PR is the one sanctioned write.

---

## Phase 1 — Preconditions

- Queue non-empty: list `csl-corrections/batch_pending/dictionaries/*/change_*.txt`. If empty, report and stop.
- Check no batch PR from us is already open: `gh pr list --repo sanskrit-lexicon/csl-orig --author @me --state open`. If one is open, stop — update that PR only if the user explicitly says so.
- Sync upstream: `git -C csl-orig checkout master && git pull` (working tree must be clean first; if it isn't, stop — something violated the read-only guard, report it).

## Phase 2 — Re-validate every queued change against current HEAD

Upstream moves between queue time and batch day. For each dict with queued changes:

1. **Verify every record against the DELIVERY BASE — `origin/main`, never the working tree.** Extract it (`git show origin/main:v02/<dict>/<dict>.txt`) and check each `<lineno> old <text>` record against it *before writing anything*. A local checkout may sit on a feature branch: on 04-08-2026 the `mw` queue verified 21,875/21,875 against the working tree and **20,618 of 21,817 failed against `origin/main`**, because five upstream commits net-deleted one line and shifted every later line number. The working tree is not the base you are shipping to.
2. **Do NOT apply the change files in sequence.** Each is prepared independently against the *pristine* dictionary, so sequential application breaks the moment one shifts or rewrites a line another targets — `ins`/`del` records shift every later anchor (`updateByLine.py`'s own docstring warns of this), and two files can rewrite the same line. Apply everything in **one pass keyed to original line numbers**: replacements by line, insertions collected per anchor. Where two files rewrite one line, merge them if their edits touch disjoint parts of it (apply the hand-list first, then any rule-shaped transformation over the whole file).
3. **Relocating onto a moved base: regenerate rules, relocate hand-lists.** A change file that encodes a *rule* (e.g. "wrap every bare `&c.`") should be **regenerated** against the new base rather than line-shifted — but only after proving the rule reproduces the approved change file **byte-for-byte on that file's own base**, and fires on exactly the same lines. A hand-list must be relocated **by content**, and content alone is not always a key: records with identical bodies can take different values, so disambiguate on the record's own `<L>` header and refuse to let two records land on one line. If a record can't be relocated (upstream already fixed it), **drop it** and log why.
4. **`updateByLine.py` is byte-unsafe on Windows** — it doubles the carriage return (+1 byte/line; measured +273,723 bytes over 273,715 ap90 lines), which rewrites *every* line in the diff. Read and write with `newline=''` instead, and sanity-check the diff size against the number of intended edits.
5. **XML-validate WITHOUT a swap window.** Do not copy the corrected text into `csl-orig` and restore it afterwards — a failed restore corrupts canonical data. Build the isolated tree first, then stage the corrected text into it:
   ```sh
   sh generate_orig.sh <dict> tempparent/<dict>
   cp <corrected>.txt tempparent/<dict>/orig/<dict>.txt
   sh generate_pywork.sh <dict> tempparent/<dict>
   (cd tempparent/<dict>/pywork && sh redo_hw.sh && sh redo_xml.sh)
   ```
   Pass = "All records parsed by ET". **Confirm the build consumed your text** (check the staged `orig/` file and the generated XML) — a green gate on pristine input proves nothing. BOM-check (no `efbbbf`).
6. **Before the PR, diff against `origin/main` and check the shape.** Replacements give a symmetric `+N -N`; an asymmetry means the base is wrong. Total diff lines must equal the intended edit count — anything more is collateral.

## Phase 3 — Branch, apply, PR

```powershell
git -C csl-orig checkout -b batch/YYYYMM-corrections
# copy each validated temp_<dict>_final.txt over v02/<dict>/<dict>.txt
git -C csl-orig add v02/ && git -C csl-orig commit -m "corrections: YYYY-MM batch (N changes across M dicts)"
git -C csl-orig push -u origin batch/YYYYMM-corrections
```

One commit per dict is acceptable if it aids review; one PR always. PR body must carry:

- A per-dict summary table: dict · lines changed · issue refs · validation result.
- `Closes #N` lines for every queued issue (this is the only issue cross-linking — no per-issue comments).
- The workflow provenance: queued via /cologne-correction-queue, validated with make_xml.py ("All records parsed by ET"), model tier+version.
- Full blob URLs for any file references.

Do NOT enable auto-merge — the maintainers merge on their own schedule; leave it `@WAITING`.

## Phase 4 — Promote the audit trail

In `csl-corrections`: move `batch_pending/` contents into `batch_YYYYMMDD/dictionaries/<dict>/` (change files + consolidated `readme.txt` documenting the workflow steps and the PR URL). Commit and push csl-corrections. Recreate an empty `batch_pending/`, dropping entries that were discarded in Phase 2 (record the drops in the readme).

## Phase 5 — Housekeeping

- `Uprava/GTD_NEXT_ACTIONS.md`: replace the "queued" line with `@WAITING` — "csl-orig batch PR #NNN awaiting maintainer merge" (full PR URL). Commit + push Uprava.
- Note for expectations: artefact regeneration is server-side (`redo_xampp_selective.sh` cron on the Cologne server) — displays update after the maintainers merge, nothing to run locally.
- Report: PR URL, dicts/lines shipped, entries dropped and why, queue now empty.

_Dr. Mārcis Gasūns_
