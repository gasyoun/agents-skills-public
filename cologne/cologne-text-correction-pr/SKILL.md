---
name: "cologne-text-correction-pr"
description: Open a source-text correction PR for a Sanskrit Lexicon issue — apply old→new via updateByLine.py, XML-validate, cross-link. Direct-PR repos only; csl-orig via /cologne-correction-queue + /cologne-batch-pr. Judgment — Sonnet 5 (claude-sonnet-5) tier. Do not use for csl-orig — that goes through /cologne-correction-queue + /cologne-batch-pr; this is for direct-PR repos only.
---

# cologne-text-correction-pr

Use this skill when the user asks to run the migrated source command `cologne-text-correction-pr`.

## Command Template


# /cologne-text-correction-pr

_Created: 03-07-2026 · Last updated: 06-09-2026_

Open a source-text correction PR for a Sanskrit Lexicon issue. Argument: `REPO ISSUE_NUMBER` (e.g. `csl-orig 1537`).

**Goal:** Read the issue, find the exact old→new text in the local source file, apply via `updateByLine.py`, XML-validate, commit to a branch, open a PR, and cross-link the issue. Never guess — if old/new text is ambiguous, post a clarification comment and stop.

## Scope — csl-orig is OUT of scope (STOP and redirect)

The direct-PR flow below (Phases 5–6) is **only** for repos that accept direct PRs — `csl-devanagari`, `csl-apidev`, dict repos, and non-dictionary-source files. **If `REPO` is `csl-orig`, or the change touches a `v02/<dict>/<dict>.txt` dictionary source, STOP here** — do not branch, push, or open a PR against csl-orig. csl-orig is read-only for direct pushes (org rule, org-level `CLAUDE.md` csl-orig section): validate + park the correction with [/cologne-correction-queue](https://github.com/gasyoun/claude-config/blob/main/commands/cologne-correction-queue.md) and ship it in the ONE consolidated ~monthly PR via [/cologne-batch-pr](https://github.com/gasyoun/claude-config/blob/main/commands/cologne-batch-pr.md). The `guard_cslorig_write.py` hook blocks a csl-orig `gh pr create` regardless — this note is so the skill stops *itself* first rather than driving into a blocked push.

---

## Autonomy rules

- Execute all phases without asking for confirmation.
- All `gh` API calls, file edits, git commits, and pushes may proceed freely.
- Windows encoding: all Python scripts include `sys.stdout.reconfigure(encoding='utf-8')`.
- **Never edit source files directly** — always go through `updateByLine.py`.
- **BOM check**: after any file write, verify first 3 bytes are not `efbbbf`.
- Retry once on transient 5xx/TLS timeouts.

---

## Phase 0 — Setup

```
ORG=sanskrit-lexicon
REPO=<first word of $ARGUMENTS>
ISSUE=<second word of $ARGUMENTS>
```

Determine source repo and source file path (Windows clone root `~/Documents/GitHub\<REPO>`;
macOS/Linux `~/Documents/GitHub/<REPO>` — same tree, forward slashes):
- `csl-orig` → `~/Documents/GitHub/csl-orig/v02/DICT/DICT.txt` (DICT = dict code from issue)
- `csl-devanagari` → `~/Documents/GitHub/csl-devanagari/v02/DICT/DICT.txt`
- `csl-apidev` → `~/Documents/GitHub/csl-apidev/`
- `csl-websanlexicon` → `~/Documents/GitHub/csl-websanlexicon/`
- `COLOGNE` → source file usually in `csl-orig/v02/DICT/` (e.g. `caeab_input.txt`)
- Other dict repos → `~/Documents/GitHub/REPO/`

---

## Phase 1 — Fetch issue and classify

```sh
gh issue view $ISSUE --repo $ORG/$REPO --json number,title,body,labels,comments
```

Read body AND all comments. Extract:
- Dictionary code (mw, shs, cae, lrv, inm, skd, etc.)
- Line number(s) or L= entry number
- Exact old text and exact new text
- Any maintainer decisions already recorded

**STOP if:**
- Old/new text is ambiguous or requires scan/scholarly judgment not supplied
- A comment says "fixed in PR#N" (already done)
- `csl-corrections` CFR/batch records this as `No change` or rejected

Check `csl-corrections` CFR for same dict+L+old text.

Windows (PowerShell):

```powershell
Select-String -Path "~/Documents/GitHub\csl-corrections\cfr_ab\cfr_ab.tsv" -Pattern "DICT.*LINENUM" | Select-Object -First 5
```

macOS / Linux (sh):

```sh
grep -m5 -E "DICT.*LINENUM" ~/Documents/GitHub/csl-corrections/cfr_ab/cfr_ab.tsv
```

---

## Phase 2 — Locate exact lines in source

Windows (PowerShell):

```powershell
# Find lines by L= entry number:
Select-String -Path "~/Documents/GitHub\csl-orig\v02\DICT\DICT.txt" -Pattern "<L>NNNN" | Select-Object LineNumber, Line

# Find lines by content:
Select-String -Path "~/Documents/GitHub\csl-orig\v02\DICT\DICT.txt" -Pattern "OLD_PATTERN" -SimpleMatch | Select-Object LineNumber, Line | Select-Object -First 10
```

macOS / Linux (sh) — `grep -n` prints `LINENUM:line`, same record:

```sh
# Find lines by L= entry number:
grep -n "<L>NNNN" ~/Documents/GitHub/csl-orig/v02/DICT/DICT.txt

# Find lines by content (first 10):
grep -n -F "OLD_PATTERN" ~/Documents/GitHub/csl-orig/v02/DICT/DICT.txt | head -10
```

Record exact line numbers and exact full line text (including any whitespace/tags).

**STOP if:** 0 matches found when expecting a match — file may already be corrected. Verify and post "appears already fixed" comment.

---

## Phase 3 — Create change file

Write to a temp scratchpad file (Windows `~/.claude/tmp\...\change_DICT_ISSUE.txt`;
macOS/Linux `~/.claude/tmp/.../change_DICT_ISSUE.txt`):

```
NNNNN old [exact full original line]
NNNNN new [exact full replacement line]
```

For multiple lines, add one `old`/`new` pair per line changed. Use `ins` for insertion (new line after), `del` for deletion.

**BOM check**: write with `open(f, 'w', encoding='utf-8')` — never `utf-8-sig`.

---

## Phase 4 — Apply and validate

Windows (PowerShell):

```powershell
# Snapshot source
Copy-Item "~/Documents/GitHub\csl-orig\v02\DICT\DICT.txt" "~/.claude/tmp\...\temp_DICT_0.txt"

# Apply
python "~/Documents/GitHub\csl-pywork\v02\makotemplates\pywork\updateByLine.py" `
    "~/.claude/tmp\...\temp_DICT_0.txt" `
    "~/.claude/tmp\...\change_DICT_ISSUE.txt" `
    "~/.claude/tmp\...\temp_DICT_1.txt"

# Verify BOM-free
python -c "
with open(r'~/.claude/tmp\...\temp_DICT_1.txt','rb') as f:
    h = f.read(3).hex()
    print('BOM' if h == 'efbbbf' else 'OK', h)
"

# Copy back
Copy-Item "~/.claude/tmp\...\temp_DICT_1.txt" `
    "~/Documents/GitHub\csl-orig\v02\DICT\DICT.txt"
```

macOS / Linux (sh) — same snapshot → apply → BOM check → copy-back shape (`$HOME`
expansion makes the paths real before python sees them):

```sh
SRC="$HOME/Documents/GitHub/csl-orig/v02/DICT/DICT.txt"
TMP="$HOME/.claude/tmp/..."

# Snapshot source
cp "$SRC" "$TMP/temp_DICT_0.txt"

# Apply
python3 ~/Documents/GitHub/csl-pywork/v02/makotemplates/pywork/updateByLine.py \
    "$TMP/temp_DICT_0.txt" \
    "$TMP/change_DICT_ISSUE.txt" \
    "$TMP/temp_DICT_1.txt"

# Verify BOM-free
python3 -c "
with open('$TMP/temp_DICT_1.txt','rb') as f:
    h = f.read(3).hex()
    print('BOM' if h == 'efbbbf' else 'OK', h)
"

# Copy back
cp "$TMP/temp_DICT_1.txt" "$SRC"
```

**XML validation** (local, without XAMPP — works identically on Windows and macOS):

Windows (PowerShell):

```powershell
cd "~/Documents/GitHub\csl-pywork\v02\makotemplates\pywork"
python make_xml.py DICT ~/.claude/tmp\...\tempparent
# Look for: "All records parsed by ET"
# Clean up: Remove-Item -Recurse "...\tempparent\DICT" -ErrorAction SilentlyContinue
```

macOS / Linux (sh) — plus an optional `xmllint` cross-check (macOS ships it; Windows does not):

```sh
cd ~/Documents/GitHub/csl-pywork/v02/makotemplates/pywork
python3 make_xml.py DICT ~/.claude/tmp/.../tempparent
# Look for: "All records parsed by ET"
xmllint --noout ~/.claude/tmp/.../tempparent/DICT/*.xml   # optional cross-check, exit 0 = well-formed
# Clean up:
rm -rf ~/.claude/tmp/.../tempparent/DICT
```

If `make_xml.py` is not accessible: use Python ElementTree directly:
```python
import xml.etree.ElementTree as ET, sys
sys.stdout.reconfigure(encoding='utf-8')
try:
    ET.parse(r'C:\path\to\output.xml')
    print('All records parsed by ET')
except ET.ParseError as e:
    print(f'PARSE ERROR: {e}')
```

**STOP if validation fails.** Revert the source file and report the error.

---

## Phase 5 — Commit to branch and open PR

Windows (PowerShell):

```powershell
cd "~/Documents/GitHub\csl-apidev"   # a direct-PR repo; NEVER csl-orig (see Scope)

# Create branch
git checkout -b "fix/DICT-issue-ISSUE"

# Stage only the changed source file
git add v02/DICT/DICT.txt  # adjust path as needed

git commit -m "fix(DICT): BRIEF_DESCRIPTION [closes REPO#ISSUE]

Co-Authored-By: Claude <noreply@anthropic.com>"

git push -u origin fix/DICT-issue-ISSUE
```

macOS / Linux (sh) — identical git flow:

```sh
cd ~/Documents/GitHub/csl-apidev   # a direct-PR repo; NEVER csl-orig (see Scope)

git checkout -b "fix/DICT-issue-ISSUE"
git add v02/DICT/DICT.txt  # adjust path as needed
git commit -m "fix(DICT): BRIEF_DESCRIPTION [closes REPO#ISSUE]

Co-Authored-By: Claude <noreply@anthropic.com>"
git push -u origin fix/DICT-issue-ISSUE
```

Open PR:
```sh
gh pr create --repo $ORG/csl-apidev \
  --title "fix(DICT): BRIEF_DESCRIPTION" \
  --body "Fixes REPO#ISSUE

**Change:** [old text] → [new text]
**Line(s):** NNNNN in v02/DICT/DICT.txt
**Validation:** All records parsed by ET

Closes $ORG/$REPO#$ISSUE"
```

Cross-link on the issue:
```sh
gh issue comment $ISSUE --repo $ORG/$REPO \
  --body "PR opened: [csl-apidev#NNN](https://github.com/sanskrit-lexicon/csl-apidev/pull/NNN)"
```

---

## Phase 6 — Audit trail (csl-corrections)

Write a change file to `csl-corrections/batch_YYYYMMDD/dictionaries/DICT/change_DICT_N.txt` and commit it separately. This is the audit trail, not applied at generation time.

---

## Phase 7 — Report

Return: issue URL, PR URL, line(s) changed, validation result, or reason for stopping.

## Change log

Newest first. This section exists so the metadata line at the top can stay two dates long.

| Date | Change |
|---|---|
| 06-09-2026 | H4201 darwin routes |

_Dr. Mārcis Gasūns_
