---
name: "cologne-markup-fix"
description: Generate a pw.txt-style markup-fixer + audit for one Cologne dictionary — inventory tags, auto-fix nested ab/whitespace, push branch, open labeled markup issue. PWG#175/PWK#113-style passes. Judgment — Sonnet 5 (claude-sonnet-5) tier. Do not use for an org-wide batch — that is /cologne-markup-batch; this generates one dictionary's markup-fixer and audit.
---

# cologne-markup-fix

Use this skill when the user asks to run the migrated source command `cologne-markup-fix`.

## Command Template


# /cologne-markup-fix

_Created: 03-07-2026 · Last updated: 05-09-2026_

Generate a `pw.txt`-style markup-fixer + audit + GitHub opening-issue comment for the Cologne dictionary repo given in the argument: $ARGUMENTS.

Counterpart skill to the work landed for PWG ([#175](https://github.com/sanskrit-lexicon/PWG/issues/175)) and PWK ([#113](https://github.com/sanskrit-lexicon/PWK/issues/113)). Execute all phases **without asking for user confirmation** unless a step is marked [ASK]. The runbook ends with a GitHub issue opened and a branch pushed.

---

## Autonomy Rules

- Execute all steps without confirmation unless marked [ASK].
- All `gh` API calls, file edits, git commits, pushes, and `gh issue create` may be executed freely.
- Retry once on transient 5xx/timeouts before continuing.
- **Windows encoding**: every Python script must include `sys.stdout.reconfigure(encoding='utf-8')` and pass `encoding='utf-8'` to any `subprocess.run`. Write multi-step scans to `.py` files rather than running inline.
- **Discover, don't hardcode**: dictionary short names (`pwg/pw/mw/…`) vary; verify the data path under `csl-orig/v02/<short>/<short>.txt` before downloading.
- **Counts come from the actual file** — never paste numbers from a precedent issue (PWG#175, PWK#113); always re-run the audit and use this dict's real counts.
- **Correction-registry preflight**: before proposing or applying any source-text change from an old issue, search `csl-corrections` CFR/batch history for the same dict/L/old/new. If it says `No change`, rejected, or deferred, stop. If accepted, decide whether the source should use a plain replacement or an inline correction layer like `{{old->new||YYYYMMDD|author|issue|}}`.

---

## Phase 0 — Setup

```
ORG=sanskrit-lexicon
REPO=<argument>          # e.g. MWS, MW72, AP, AP90, SKD, VCP
```

Verify access and fetch CLAUDE.md (if present) to identify the data file:
```sh
gh api repos/$ORG/$REPO --jq '{name, description}'
gh api repos/$ORG/$REPO/contents/CLAUDE.md --jq '.content' 2>/dev/null | base64 -d 2>/dev/null | head -40
```

**Resolve `csl-orig` short name** for the dictionary. CLAUDE.md usually names it (e.g. "`csl-orig/v02/mw/mw.txt`"). If absent, try the obvious candidates in order: lowercase repo name with `S` suffix dropped (`MWS → mw`), then full lowercase (`MWS → mws`). Verify via:
```sh
gh api repos/$ORG/csl-orig/contents/v02/<short> --jq '.[].name' | grep -E '^<short>\.txt$'
```

Record `SHORT` and the data URL `https://raw.githubusercontent.com/$ORG/csl-orig/master/v02/$SHORT/$SHORT.txt`.

Create the local workspace:
```sh
mkdir -p D:/claude/${SHORT}_markup_fix
cd       D:/claude/${SHORT}_markup_fix
curl -s -L -o $SHORT.txt <data url>
wc -l $SHORT.txt
```

---

## Phase 1 — Tag inventory

Write `scan_tags.py` (inline below; this is the only "scan" file kept short-lived — it does NOT get committed). It reports paired-tag counts, self-closing tags, attribute usage on `<ab n="…">`, and the four oddity classes we audit.

```python
import re, sys
from pathlib import Path
from collections import Counter
sys.stdout.reconfigure(encoding="utf-8")

TXT = Path(__file__).resolve().parent / "<SHORT>.txt"
text = TXT.read_text(encoding="utf-8")
lines = text.splitlines()
print(f"lines: {len(lines):,}   chars: {len(text):,}")

closes = Counter(re.findall(r"</([A-Za-z][A-Za-z0-9_]*)>", text))
opens  = Counter(re.findall(r"<([A-Za-z][A-Za-z0-9_]*)\b[^>]*>", text))
print("\nclosing-tag counts (balanced pairs only):")
for t, n in closes.most_common(20):
    if opens.get(t, 0) == n:
        print(f"  </{t}>: {n:>9,}")
    else:
        print(f"  </{t}>: {n:>9,}  (UNBALANCED — opens {opens.get(t,0)})")

self_close = Counter(re.findall(r"<([A-Za-z][A-Za-z0-9_]*)/>", text))
print("\nself-closing tags:")
for t, n in self_close.most_common(10):
    print(f"  <{t}/>: {n}")

# Universal audits
print("\nnested <ab>:               ", len(re.findall(r"<ab\b[^>]*>[^<]*<ab\b", text)))
print("<ab n='?'> placeholders:   ", len(re.findall(r'<ab\s+n="\?">', text)))
print("adjacent </ab> <ab>:       ", len(re.findall(r"</ab>\s*<ab\b", text)))
print("nested <ls>:               ", len(re.findall(r"<ls\b[^>]*>[^<]*<ls\b", text)))
print("{%…%}<is> collisions:      ", len(re.findall(r"%\}<is\b", text)))
print("{#…#}<ab|ls|is> collisions:", len(re.findall(r"#\}<(?:ab|ls|is)\b", text)))
print("</ls>.[Page glued:         ", len(re.findall(r"</ls>\.\[Page\d", text)))

# Whitespace inside paired tags
print("\nwhitespace inside paired tags (lead-space / trail-space):")
for tag in [t for t, n in closes.most_common(20) if opens.get(t, 0) == n]:
    lead  = len(re.findall(rf"<{tag}\b[^>]*> [^<\s]", text))
    trail = len(re.findall(rf"[^<\s] </{tag}>", text))
    if lead or trail:
        print(f"  <{tag}>: lead={lead}, trail={trail}")

# Correction-record format detection
cb = len(re.findall(r"\{\{[^{}]*->[^{}]*\|\|", text))
print(f"\n{{{{old -> new || …}}}} correction records: {cb}")

# <ab n="…"> attribute population
ab_n = re.findall(r'<ab\s+n="([^"]+)">', text)
print(f"\n<ab n='…'> attribute count: {len(ab_n)}, unique values: {len(set(ab_n))}")
if ab_n:
    for v, n in Counter(ab_n).most_common(5):
        print(f"  n=\"{v}\": {n}")
```

Run it. **Record the closing-tag inventory and oddity counts** — they go into both the audit and the issue comment.

---

## Phase 2 — Identify dictionary-specific patterns

Cross-reference findings against the precedent table:

| Pattern | PWG | PWK | MW(S) | Likely action |
|---|---|---|---|---|
| Nested `<ab>` | 0 | 0 | ? | Auto-fix (regex `NEST_RX`) |
| `<ab n="?">` placeholders | 91 | 0 | ? | Audit, do not auto-fix |
| Nested `<ls>` in `{{…}}` corrections | n/a | 2 | ? | Audit, classify by context |
| Adjacent `</ab> <ab>` | 9 | 4,171 | ? | Audit, do not auto-merge |
| `{%…%}<is>` collision | n/a | 1 | ? | Audit singleton |
| `</ls>.[Page…]` glue | n/a | 1 | ? | Audit singleton |
| Trim-eligible tag pairs | 5 | 12 | ? | Build `TRIM_TAGS` list |

For tags present in this dict but absent from the table above, decide:
- If usage > 0 and the tag has paired open/close → add to `TRIM_TAGS`.
- If usage < 30 → mention in "PWK-specific tag content"-style note for visual review.
- If the tag's purpose is unclear → [ASK] the user once, batching all unclear tags.

If any audit finding is tied to a pre-existing correction issue rather than a
pure markup pattern, run the correction-registry preflight before turning it into
an edit. Do not let an open old issue override a `No change` registry status.

---

## Phase 3 — Generate `08_markup_fix.py`

Template lives at PWK [pwkissues/markup_fix/08_markup_fix.py](https://github.com/sanskrit-lexicon/PWK/blob/main/pwkissues/markup_fix/08_markup_fix.py) (the original `markup-fix-audit` branch was merged and deleted — master carries the file). Customise:

1. **Module docstring** — replace `pw.txt` / `PWK` with this dict's filename and repo.
2. **`PW_TXT` default path** — point at `csl-orig/v02/$SHORT/$SHORT.txt`.
3. **`TRIM_TAGS`** — restrict to tag names with non-zero balanced open/close in Phase 1.
4. **`AUDIT_CHECKS`** — keep universal rows (nested `<ab>`, placeholder, empty tag, malformed). Adjust the two boundary-collision rows to the tags actually present (e.g. drop `{%…%}<is>` if no `<is>` exists in this dict). Add nested-`<ls>`-inside-`{{…}}` only if Phase 1 detected `{{…->…||}}` correction records.
5. **`_ls_nested_classify`** — keep only if correction records were detected; otherwise the simpler "Nested `<ls>`" check is enough.
6. **Audit prose** — the "WHAT NEEDS HUMAN ATTENTION" block must be rewritten to match this dict's actual findings, not PWK's.

**Always write the synthetic test file** `test_markup_fix.py` covering: nested `<ab>` (prefix, suffix, exact-dup, middle-leave-alone), whitespace trim for two or three of this dict's most common paired tags, and one no-op case. Run it before continuing — block on 0 failures.

---

## Phase 4 — Run the fixer

```sh
python 08_markup_fix.py
python test_markup_fix.py    # must report N/N passed
```

Verify outputs:
- `<short>_fixed.txt` exists and is non-empty.
- `markup_fix_changes.txt` is the log (may be empty if no auto-fixes triggered).
- `markup_audit.txt` lists all audit rows with hit counts and ≤200 sample lines per row.

**Sha-verify byte-identity** of the fixed file vs source when zero auto-fixes occurred:
```sh
python -c "import hashlib; a=hashlib.sha256(open('<short>.txt','rb').read()).hexdigest(); b=hashlib.sha256(open('<short>_fixed.txt','rb').read()).hexdigest(); print('IDENTICAL' if a==b else 'DIFFERENT')"
```

If `IDENTICAL`, mention this in the comment ("output is byte-identical to source").

---

## Phase 5 — Compose the GitHub issue body

Write `ISSUE_COMMENT.md` with these sections, in order:

1. **`### Location`** — opening paragraph referencing PWG#175 and PWK#113 as precedents, naming the branch where files will live.
2. **`## Markup fixer + audit for <short>.txt`** — section header.
3. **`### What it auto-fixes`** — markdown table of fix patterns. Always include the four canonical examples (nested `<ab>`, three variants) plus 1–2 whitespace examples drawn from this dict's actual `TRIM_TAGS`.
4. **`### Closing-tag inventory in current <short>.txt`** — table of all balanced paired tags with counts (from Phase 1). Include self-closing tags as a final row if any exist.
5. **`### What it found in current <short>.txt`** — bulleted list of the seven canonical findings with **real counts** from Phase 4. Order: nested `<ab>`, whitespace trims, `<ab n="?">` placeholders, nested `<ls>`, boundary singletons, adjacent `</ab> <ab>`. Include 2–3 sample line numbers per non-trivial finding.
6. **`### Broader cleanup checklist (in markup_audit.txt)`** — six numbered items mirroring the audit's "WHAT NEEDS HUMAN ATTENTION" prose.
7. **`### Usage`** — `cd pwkissues/markup_fix; python 08_markup_fix.py` block (path adjusted to this repo's `<repo>issues/markup_fix/`).
8. **`### Summary`** — 4-line one-paragraph close: tag-balanced? auto-fixes needed? boundary collisions? Plus the non-zero finding list.
9. **`### Severity`** — `minor` for ≤ 5 boundary singletons + audit-only finds; `medium` if real fixes are needed; `hard` if structural problems.

@-mention `@funderburkjim @Andhrabharati` in the Location paragraph.

Save a second copy as `comment_markup_fix.md` (older PWG-style name) for repo-internal reference.

---

## Phase 6 — Stage and push

Clone the target repo into `D:/claude/<repo lowercase>_repo` if not already present:
```sh
cd D:/claude
git clone https://github.com/sanskrit-lexicon/$REPO.git ${REPO_LOWER}_repo
```

(If a repo-name-cased directory already exists in `D:/claude/sl-batch/$REPO`, use it instead — keeps clones consolidated.)

Branch + copy:
```sh
cd ${REPO_LOWER}_repo
git checkout -b markup-fix-audit
mkdir -p <repo lowercase>issues/markup_fix
cp ../${SHORT}_markup_fix/{08_markup_fix.py,test_markup_fix.py,markup_audit.txt,markup_fix_changes.txt,${SHORT}_fixed.txt,comment_markup_fix.md,ISSUE_COMMENT.md,readme.txt} <repo lowercase>issues/markup_fix/
```

The `<repo lowercase>issues/` directory mirrors how PWG uses `pwgissues/` and PWK uses `pwkissues/`. For repos that don't follow this convention (e.g. MWS uses `mwsissues/`? verify by `ls` in the repo first), fall back to whatever existing `*issues/` directory the repo already has.

Add a 3-line `readme.txt`:
```
Counterpart of PWG/pwgissues/issue174/08_markup_fix.py
(see https://github.com/sanskrit-lexicon/PWG/issues/175
 and https://github.com/sanskrit-lexicon/PWK/issues/113).
…run instructions and file list…
```

Commit and push (no force):
```sh
git add <repo lowercase>issues/markup_fix/
git commit -m "Add markup fixer + audit for $SHORT.txt

Counterpart of PWG#175 and PWK#113. Analyzes $SHORT.txt for markup
oddities (auto-fixes nested <ab> + whitespace; audits everything else
with line refs). N/N synthetic tests pass.

Co-Authored-By: Claude <executing tier + version, e.g. Sonnet 5 (claude-sonnet-5)> <noreply@anthropic.com>"
git push -u origin markup-fix-audit
```

---

## Phase 7 — Open the GitHub issue

```sh
gh issue create \
  --repo $ORG/$REPO \
  --title "[markup] Minor $SHORT.txt Markup Oddities" \
  --body "$(cat <repo lowercase>issues/markup_fix/ISSUE_COMMENT.md)" \
  -l markup -l minor
```

Capture the returned URL. If labels `markup` / `minor` don't exist in this repo (some tooling repos lack them), drop the `-l` flags and tell the user the issue is unlabeled.

---

## Phase 8 — Verification

```sh
gh issue view <returned number> --repo $ORG/$REPO --json number,title,state,labels,url
```

Confirm:
- Title matches `[markup] Minor <short>.txt Markup Oddities`.
- State is `OPEN`.
- Labels include `markup` (if available in the repo).
- URL resolves.

Print to the user:
- Issue URL.
- Branch URL (`https://github.com/$ORG/$REPO/tree/markup-fix-audit/<repo lowercase>issues/markup_fix`).
- One-line findings summary (e.g. "0 auto-fixes, 2 boundary singletons, 4,171 adjacent `</ab> <ab>` for review").

---

## Checklist

- [ ] Data file located in `csl-orig/v02/<short>/<short>.txt` and downloaded.
- [ ] Tag inventory recorded (closing counts table).
- [ ] `08_markup_fix.py` generated with the dict's actual `TRIM_TAGS`.
- [ ] Synthetic tests pass (N/N).
- [ ] Audit produced, `<short>_fixed.txt` written.
- [ ] `ISSUE_COMMENT.md` uses **this dict's real counts**, not PWG's or PWK's.
- [ ] Branch pushed.
- [ ] Issue opened, URL captured, labels applied.
- [ ] Final summary sent to user includes issue URL, branch URL, and one-line findings.

---

## Notes carried over from PWG / PWK runs

- Adjacent `</ab> <ab>` is **almost always** intentional ("`<ab>Pl.</ab> <ab>Bez.</ab>`"). Audit-only; do not auto-merge.
- Nested tags inside `{{old -> new || date | author | URL |}}` are the correction-record format, not a markup bug. Classify them as informational.
- `pw_fixed.txt`-style outputs that are byte-identical to source ARE valuable to commit — they fix the diff baseline for future re-runs after wrap/overlay passes.
- The `<ab n="?">` audit row is worth keeping even when the count is 0 — a future auto-wrap pass may introduce them.

_Dr. Mārcis Gasūns_
