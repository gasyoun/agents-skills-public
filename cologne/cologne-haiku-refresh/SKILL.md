---
name: "cologne-haiku-refresh"
description: Run the Round-2 Haiku maintenance pass over the MWS docs-pass papers/microanalysis/ tree — refresh palette/figures/manifest in dependency order, audit verifiers, push date-stamped branch. Mechanical maintenance — Haiku 4.5 (claude-haiku-4-5) tier. Do not use for any repo outside the MWS docs-pass papers/microanalysis tree, and not for dictionary headword triage — that is /cologne-dict-triage; this is a narrow, scoped maintenance refresh.
---

# cologne-haiku-refresh

Use this skill when the user asks to run the migrated source command `cologne-haiku-refresh`.

## Command Template


_Created: 03-07-2026 · Last updated: 01-10-2026_

Run the Round-2 Haiku maintenance pass (H1–H10) over the MWS `docs-pass` branch's `papers/microanalysis/` tree. Refresh data + figures in dependency order, audit verifiers, push a branch, and open a GitHub issue summarising the refresh. Argument is the date stamp to use (defaults to today): $ARGUMENTS.

Counterpart to [`cologne-markup-fix`](https://github.com/gasyoun/claude-config/blob/main/commands/cologne-markup-fix.md) but for the Round-2 microanalysis pipeline described in [HANDOFF_PROMPTS.md](https://github.com/sanskrit-lexicon/MWS/blob/docs-pass/papers/microanalysis/HANDOFF_PROMPTS.md). Haiku is the **optional first tier** before the Sonnet 5 (claude-sonnet-5) JSON exports and the Opus 4.8 (claude-opus-4-8) chapter authoring — this skill executes that tier end-to-end.

---

## Autonomy Rules

- Execute all steps without confirmation unless marked [ASK].
- Hard-stop steps (H5, H1, H7) must succeed before later steps run — they produce artifacts everything else verifies.
- Soft-warn steps (H4, H3, H2, H6, H8, H9, H10) record findings into the audit table but **do not halt** the run.
- All `gh` API calls, file edits, git commits, pushes, and `gh issue create` may be executed freely.
- Retry once on transient 5xx/timeouts before continuing.
- **Windows encoding**: every Python script must include `sys.stdout.reconfigure(encoding='utf-8')` and pass `encoding='utf-8'` to any `subprocess.run`. Write multi-step scans to `.py` files rather than running inline.
- **Discover, don't hardcode**: the 9 dictionaries' short names live in the renderer scripts themselves; do not duplicate the list here.
- **Counts come from the actual files** — never paste numbers from a prior refresh run; always re-derive.
- **Issue body links**: every filename mention in the issue body (tables, prose, checklists) must be a hyperlink — no bare backtick filenames. See [Issue body link rule memory](file:///~/.claude/projects/D--claude/memory/feedback_issue_links.md).

---

## Phase 0 — Setup

```
ORG=sanskrit-lexicon
REPO=MWS
BRANCH_BASE=docs-pass
REFRESH_DATE=${ARGUMENTS:-$(date +%Y-%m-%d)}
WORK_BRANCH=haiku-refresh-$REFRESH_DATE
WORKSPACE=D:/claude/mws_haiku_refresh_$REFRESH_DATE
```

Clone or reuse the consolidated clone:

```sh
mkdir -p D:/claude
cd       D:/claude
if [ -d sl-batch/MWS ]; then
  cd sl-batch/MWS && git fetch origin && git checkout $BRANCH_BASE && git pull --ff-only
else
  git clone -b $BRANCH_BASE https://github.com/$ORG/$REPO.git mws_repo
  cd mws_repo
fi

git checkout -b $WORK_BRANCH
MWS_ROOT=$(pwd)
MA=$MWS_ROOT/papers/microanalysis
ls $MA/figures/scripts/ $MA/analysis/ $MA/figures/locales/
```

Verify the four key script paths exist (fail fast otherwise):

```sh
test -f $MA/figures/scripts/build_palette.py
test -f $MA/figures/scripts/render_heatmap.py
test -f $MA/figures/scripts/render_treemap.py
test -f $MA/figures/scripts/render_sankey.py
test -f $MA/analysis/make_supplement.py
```

Layout verified against `docs-pass` on 2026-05-28. All four renderer + palette scripts live in `figures/scripts/` (not `figures/` top-level as HANDOFF_PROMPTS implies). If any test fails on a future run, [ASK] the user — the branch may have reorganised again.

Create the audit log file the later phases append to:

```sh
mkdir -p $WORKSPACE
AUDIT=$WORKSPACE/haiku_refresh_audit.md
printf "# Round-2 Haiku refresh — %s\n\n| Step | Status | Findings |\n|---|---|---|\n" "$REFRESH_DATE" > $AUDIT
```

---

## Phase 1 — H5 palette consistency (hard-stop)

Palette must be rebuilt **before** renderers so figures pick up current tokens. Outputs land in `$MA/figures/` (parent of `scripts/`).

```sh
cd $MA/figures/scripts
python build_palette.py
```

Verify the three outputs all changed coherently:

```sh
cd $MA/figures
git diff --stat palette-tokens.json palette.css palette.py
```

If only one of the three changed, the palette source-of-truth has drifted — abort and [ASK].

Append to audit:

```
| H5 palette | ✅ | <N> tokens, 3 outputs regenerated |
```

---

## Phase 1.5 — Conditional data refresh (export scripts)

Three scripts in `figures/scripts/` produce the JSON inputs the renderers consume: `export_data.py` (per-block matrices), `export_dict_blocks.py` (per-dict block JSONs), `export_cross_dict.py` (`cross-dict.json`). They are **idempotent** — running them when `mw.txt` hasn't changed produces byte-identical output to what's already committed.

Run them **only** when upstream `mw.txt` (at `~/.claude/tmp/mw.txt`) is newer than the data JSONs in `$MA/figures/data/`. Otherwise they're a slow no-op (large file parse):

```sh
MW_TXT=~/.claude/tmp/mw.txt
NEWEST_JSON=$(ls -t $MA/figures/data/*.json 2>/dev/null | head -1)

if [ -f "$MW_TXT" ] && [ "$MW_TXT" -nt "$NEWEST_JSON" ]; then
  echo "mw.txt is newer than data JSONs — refreshing..."
  cd $MA/figures/scripts
  python export_data.py        2>&1 | tee $WORKSPACE/h1_5_export_data.log
  python export_dict_blocks.py 2>&1 | tee $WORKSPACE/h1_5_export_blocks.log
  python export_cross_dict.py  2>&1 | tee $WORKSPACE/h1_5_export_cross.log
  printf "| H1.5 data refresh | ✅ | mw.txt newer; 3 export scripts re-run |\n" >> $AUDIT
else
  echo "mw.txt unchanged — skipping data refresh"
  printf "| H1.5 data refresh | ✅ | skipped (mw.txt unchanged) |\n" >> $AUDIT
fi
```

If the `mw.txt` TMP path doesn't exist on the runner machine, skip silently and note `| H1.5 data refresh | ℹ️ | skipped (mw.txt not available locally) |`.

---

## Phase 2 — H1 renderers (hard-stop)

Re-run **all four** figure renderers for **both locales** where supported. The renderers default to `--locale en`; ru must be invoked explicitly. `render_cross_dict.py` is en-only by design (ru deferred per [DOUBTS](https://github.com/sanskrit-lexicon/MWS/blob/docs-pass/papers/microanalysis/DOUBTS.md) D11). Outputs (PNG + SVG + `.alt.txt` + `.desc.txt`) land in `$MA/figures/`.

```sh
cd $MA/figures/scripts
# en locale (default)
python render_heatmap.py            2>&1 | tee $WORKSPACE/h1_heatmap_en.log
python render_treemap.py            2>&1 | tee $WORKSPACE/h1_treemap_en.log
python render_sankey.py             2>&1 | tee $WORKSPACE/h1_sankey_en.log
python render_cross_dict.py         2>&1 | tee $WORKSPACE/h1_cross_dict.log
# ru locale (explicit)
python render_heatmap.py --locale ru 2>&1 | tee $WORKSPACE/h1_heatmap_ru.log
python render_treemap.py --locale ru 2>&1 | tee $WORKSPACE/h1_treemap_ru.log
python render_sankey.py  --locale ru 2>&1 | tee $WORKSPACE/h1_sankey_ru.log
```

Any non-zero exit is a hard stop. Capture the per-renderer file count for the audit:

```sh
HEAT=$(ls $MA/figures/*heatmap*.png 2>/dev/null | wc -l)
TREE=$(ls $MA/figures/*treemap*.png 2>/dev/null | wc -l)
SANK=$(ls $MA/figures/*sankey*.png  2>/dev/null | wc -l)
CD=$(ls   $MA/figures/cross-dict-*.png 2>/dev/null | wc -l)
```

Append to audit:

```
| H1 renderers | ✅ | heatmap=<HEAT>, treemap=<TREE>, sankey=<SANK>, cross-dict=<CD> (en+ru where supported; cross-dict ru deferred per DOUBTS D11) |
```

**Common pitfall**: skipping `--locale ru` leaves the ru figures from a prior commit, which then fail H4's SHA check. Always run both locales.

---

## Phase 3 — H4 version-stamp verification (soft-warn)

Renderers wrote the footer; we only verify it's present. **Three footer formats coexist** — the verifier must accept all three:

- Format A (heatmap/treemap/sankey en): `Source: CDSL <short>.txt YYYY-MM-DD · CC-BY-SA-4.0 · build <SHA>`
- Format B (cross-dict en):              `Source: CDSL YYYY-MM-DD · CC-BY-SA-4.0 · build <SHA>` (no `<short>.txt`)
- Format C (heatmap/treemap/sankey ru):  `Источник: CDSL <short>.txt YYYY-MM-DD · CC-BY-SA-4.0 · сборка <SHA>` (Cyrillic prefix + "сборка")

"Stale SHA" is **informational, not a warning** — figures rendered at commit X naturally diverge from HEAD after commit X+1. Only "missing footer" is a real ⚠️.

Write `$WORKSPACE/verify_stamp.py`:

```python
"""Locale-agnostic, format-agnostic footer-stamp verifier."""
import re, sys, subprocess
from pathlib import Path
sys.stdout.reconfigure(encoding='utf-8')

FIG = Path(sys.argv[1])
SHA = subprocess.check_output(['git','rev-parse','--short','HEAD'],
                              cwd=str(FIG.parent.parent.parent),
                              encoding='utf-8').strip()
# Match any prefix word + ': CDSL' + optional <short>.txt + date +
# ' · CC-BY-SA-4.0 · ' + any build-word + SHA. Works for en/ru/cross-dict.
pat = re.compile(
    r"\S+: CDSL [^·]*?\d{4}-\d{2}-\d{2} · CC-BY-SA-4\.0 · \S+ ([0-9a-f]+)",
    re.UNICODE,
)

missing, stale = [], []
svgs = sorted(FIG.glob('*.svg'))
for svg in svgs:
    text = svg.read_text(encoding='utf-8', errors='ignore')
    m = pat.search(text)
    if not m:
        missing.append(svg.name)
    elif m.group(1)[:7] != SHA[:7]:
        stale.append((svg.name, m.group(1)))

print(f"checked: {len(svgs)}")
print(f"missing footer: {len(missing)}  (real warning)")
print(f"stale SHA:      {len(stale)}     (informational — figures rendered at earlier commit)")
print(f"current HEAD:   {SHA}")
for n in missing[:8]: print(f"  missing: {n}")
for n, sha in stale[:8]: print(f"  stale:   {n} (build {sha})")
```

Run and record:

```sh
python $WORKSPACE/verify_stamp.py $MA/figures | tee $WORKSPACE/h4_stamp.log
```

Append to audit (status = ✅ if both counts are 0, else ⚠️ with counts).

---

## Phase 4 — H3 PNG↔SVG parity (soft-warn)

Every figure should exist in both formats. List discrepancies — do not regenerate (Phase 2 already did).

Note: `analysis/check_figure_consistency.py` exists in the repo but it audits **cross-figure data consistency** (record counts across the JSON inputs), not PNG↔SVG parity. Use the dedicated script below instead.

Write `$WORKSPACE/png_svg_parity.py`:

```python
import sys
from pathlib import Path
sys.stdout.reconfigure(encoding='utf-8')

FIG = Path(sys.argv[1])
pngs = {p.stem for p in FIG.glob('*.png')}
svgs = {p.stem for p in FIG.glob('*.svg')}
only_png = sorted(pngs - svgs)
only_svg = sorted(svgs - pngs)
print(f"PNG only ({len(only_png)}): {only_png[:10]}")
print(f"SVG only ({len(only_svg)}): {only_svg[:10]}")
```

Run, capture counts, append to audit:

```sh
python $WORKSPACE/png_svg_parity.py $MA/figures | tee $WORKSPACE/h3_parity.log
```

---

## Phase 5 — H2 alt/desc sidecar audit (soft-warn)

Every figure must have matching `.alt.txt` and `.desc.txt`. List gaps; do not auto-generate (renderers own that).

Write `$WORKSPACE/sidecar_audit.py`:

```python
import sys
from pathlib import Path
sys.stdout.reconfigure(encoding='utf-8')

FIG = Path(sys.argv[1])
bases = sorted({p.stem for p in FIG.glob('*.png')} | {p.stem for p in FIG.glob('*.svg')})
miss_alt  = [b for b in bases if not (FIG / f"{b}.alt.txt").exists()]
miss_desc = [b for b in bases if not (FIG / f"{b}.desc.txt").exists()]
print(f"figures: {len(bases)}")
print(f"missing .alt.txt:  {len(miss_alt)}")
print(f"missing .desc.txt: {len(miss_desc)}")
for n in miss_alt[:10]:  print(f"  alt:  {n}")
for n in miss_desc[:10]: print(f"  desc: {n}")
```

Append to audit.

---

## Phase 6 — H6 locale string parity (soft-warn)

Key sets in `en.json` and `ru.json` must match. Audit only — translation is not Haiku's job.

```sh
python -c "
import json, sys
sys.stdout.reconfigure(encoding='utf-8')
en = json.load(open(r'$MA/figures/locales/en.json', encoding='utf-8'))
ru = json.load(open(r'$MA/figures/locales/ru.json', encoding='utf-8'))
en_keys, ru_keys = set(en), set(ru)
print(f'en={len(en_keys)} ru={len(ru_keys)}')
print(f'en-only ({len(en_keys-ru_keys)}):', sorted(en_keys-ru_keys)[:10])
print(f'ru-only ({len(ru_keys-en_keys)}):', sorted(ru_keys-en_keys)[:10])
" | tee $WORKSPACE/h6_locales.log
```

Append to audit.

---

## Phase 7 — H7 manifest regen (hard-stop)

Regenerate `SUPPLEMENT_MANIFEST.md` and the supplementary ZIP.

```sh
cd $MA/analysis
python make_supplement.py 2>&1 | tee $WORKSPACE/h7_manifest.log
```

Capture the new manifest file count and ZIP size for the audit. The script produces `mw-microanalysis-supplementary.zip` alongside `SUPPLEMENT_MANIFEST.md`:

```sh
# Parse the manifest's own "**N files**" header rather than grepping bullets —
# the section summaries (e.g. "- **analysis/** — 24 files") have leading dashes
# and would inflate a naive count.
MAN_FILES=$(grep -oE '\*\*[0-9]+ files\*\*' $MA/analysis/SUPPLEMENT_MANIFEST.md | grep -oE '[0-9]+' | head -1)
ZIP=$MA/analysis/mw-microanalysis-supplementary.zip
test -f "$ZIP" || { echo "ZIP not produced — make_supplement.py may have errored"; exit 1; }
ZIP_SIZE=$(stat -c%s "$ZIP" 2>/dev/null || wc -c < "$ZIP")
```

---

## Phase 8 — H8 ZIP integrity (soft-warn)

Unzip to a scratch dir, count entries, verify the count matches the manifest's filelist.

```sh
mkdir -p $WORKSPACE/zip_check
unzip -q -o "$ZIP" -d $WORKSPACE/zip_check
ZIP_COUNT=$(find $WORKSPACE/zip_check -type f | wc -l)
echo "manifest files: $MAN_FILES   zip files: $ZIP_COUNT"
```

Mismatch → ⚠️ row with both counts. Match → ✅ row with the matched count.

---

## Phase 9 — H9 CI status audit (soft-warn)

```sh
gh run list -R $ORG/$REPO         --limit 5 --json status,conclusion,name,createdAt,url,databaseId
gh run list -R $ORG/csl-atlas     --limit 5 --json status,conclusion,name,createdAt,url,databaseId
```

Tally `success` / `failure` / `in_progress` per repo. **For every `failure`, fetch the actual error** — never just report "1 failure" without identifying the root cause:

```sh
FAILED_ID=$(gh run list -R $ORG/$REPO --limit 5 --status failure --json databaseId --jq '.[0].databaseId')
if [ -n "$FAILED_ID" ]; then
  gh run view $FAILED_ID -R $ORG/$REPO --log-failed 2>&1 \
    | grep -iE "##\[error\]|^Error:|Liquid|fatal" \
    | head -5
fi
```

The ⚠️ row must name the root cause (e.g. "Liquid syntax error in `DATA_DICTIONARY.md`") and state whether it's microanalysis-related or not. A vague "1 failure" is not acceptable.

---

## Phase 10 — H10 cross-repo link-rot scan (soft-warn)

## Phase 11 — Compose the GitHub issue body

## Phase 12 — Stage and push

## Phase 13 — Open the GitHub issue

Ensure the labels exist (create silently if missing — `refresh` + `maintenance` are now established MWS labels per the 2026-05-28 run):

```sh
gh label create refresh     --repo $ORG/$REPO --description "Data refresh/maintenance pass" --color "FEF2C0" 2>/dev/null || true
gh label create maintenance --repo $ORG/$REPO --description "Maintenance task"             --color "BFDADC" 2>/dev/null || true

gh issue create \
  --repo $ORG/$REPO \
  --title "[refresh] Round-2 Haiku maintenance — $REFRESH_DATE" \
  --body "$(cat mwsissues/haiku_refresh/$REFRESH_DATE/ISSUE_COMMENT.md)" \
  -l refresh -l maintenance
```

Capture the returned URL.

---

## Phase 14 — Verification

```sh
gh issue view <returned number> --repo $ORG/$REPO --json number,title,state,labels,url
```

Confirm:
- Title matches `[refresh] Round-2 Haiku maintenance — $REFRESH_DATE`.
- State is `OPEN`.
- URL resolves.

Print to the user:
- Issue URL.
- Branch URL (`https://github.com/$ORG/$REPO/tree/$WORK_BRANCH/papers/microanalysis/figures`).
- One-line summary: `"H5/H1/H7 ✅, audits: <K> warnings across H<list>"`.

---

## Checklist

- [ ] `$WORK_BRANCH` created from `docs-pass`.
- [ ] H5 palette: 3 outputs regenerated coherently.
- [ ] H1.5 data refresh: conditional on mw.txt mtime (or marked skipped if mw.txt unavailable).
- [ ] H1 renderers: all FOUR scripts run (heatmap, treemap, sankey, cross-dict) for BOTH locales where supported (cross-dict ru deferred per DOUBTS D11).
- [ ] H4 footer stamps verified with locale-agnostic regex (accepts `Source:` + `Источник:` + cross-dict-style without `<short>.txt`).
- [ ] H3 PNG↔SVG parity recorded (zero drift expected).
- [ ] H2 alt/desc sidecars audited (zero gaps expected).
- [ ] H6 locale key-set diff recorded.
- [ ] H7 `SUPPLEMENT_MANIFEST.md` regenerated; ZIP rebuilt but NOT git-added (gitignored).
- [ ] H7 manifest count parsed from `**N files**` header — not by grepping bullets.
- [ ] H8 ZIP file-count matches manifest count.
- [ ] H9 last 5 CI runs recorded for MWS and csl-atlas; **every `failure` has its root cause fetched via `gh run view --log-failed`**.
- [ ] H10 real HEAD-check on URLs (not just URL counting); log committed to `mwsissues/haiku_refresh/$REFRESH_DATE/h10_link_rot.log`.
- [ ] `ISSUE_COMMENT.md` filenames are all hyperlinks (no bare backticks); ZIP path is mentioned as bare text, NOT linked.
- [ ] Labels `refresh` + `maintenance` created-if-missing before `gh issue create`.
- [ ] Branch pushed, issue opened, URL captured.

---

## Notes carried over from Round-2 design
## Lessons from the 2026-05-28 run (issue [#197](https://github.com/sanskrit-lexicon/MWS/issues/197), gap analysis in [GAPS.md](https://github.com/sanskrit-lexicon/MWS/blob/haiku-refresh-2026-05-28/mwsissues/haiku_refresh/2026-05-28/GAPS.md))
_Dr. Mārcis Gasūns_
## References

Detail moved verbatim (read on demand — lives in the rendered skill copy's references/):

- `references/phase10-link-rot.md` — H10 cross-repo link-rot scan procedure
- `references/phase11-issue-body.md` — GitHub issue body composition for the refresh report
- `references/phase12-push.md` — Stage and push procedure
- `references/notes-carried-over.md` — Notes carried over from Round-2 design
- `references/lessons-learned.md` — Lessons from the 2026-05-28 run (MWS#197, GAPS.md)
