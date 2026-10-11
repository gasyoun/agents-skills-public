Write `$WORKSPACE/ISSUE_COMMENT.md` with these sections in order:

1. **`### Location`** — opening paragraph referencing this is a Round-2 Haiku-tier refresh per [HANDOFF_PROMPTS.md](https://github.com/sanskrit-lexicon/MWS/blob/docs-pass/papers/microanalysis/HANDOFF_PROMPTS.md), naming `$WORK_BRANCH`. @-mention `@funderburkjim @Andhrabharati`.
2. **`## Round-2 Haiku refresh — $REFRESH_DATE`** — section header.
3. **`### Audit table`** — copy `$AUDIT` table verbatim. Every filename in the Findings column must be a clickable link (per the [Issue body link rule](file:///~/.claude/projects/D--claude/memory/feedback_issue_links.md)).
4. **`### Artifacts refreshed`** — bulleted list of paths touched, with link to each on the `$WORK_BRANCH`:
   - `papers/microanalysis/figures/palette-tokens.json` + `palette.css` + `palette.py`
   - `papers/microanalysis/figures/*.png` + `*.svg` + `*.alt.txt` + `*.desc.txt`
   - `papers/microanalysis/analysis/SUPPLEMENT_MANIFEST.md`
   - The supplementary `.zip` is **rebuilt but not committed** — `analysis/.gitignore` excludes `*.zip` by repo policy. Do NOT hyperlink the ZIP filename (it 404s on GitHub); mention the path as bare text with a one-line rebuild instruction: "rebuild locally with `python papers/microanalysis/analysis/make_supplement.py`".
5. **`### Findings requiring attention`** — only the ⚠️ rows from the audit, each with sample line refs / URLs / filenames.
6. **`### Usage`** — single-line reproduction:
   ```
   /cologne-haiku-refresh $REFRESH_DATE
   ```
7. **`### Severity`** — `minor` if every step is ✅ or only H10 has ⚠️; `medium` if any of H3/H2/H6/H8/H9 ⚠️; `hard` if H5/H1/H7 failed (in which case the issue won't be opened — the run aborted earlier).

Save a second copy as `comment_haiku_refresh.md` for repo-internal reference.

---

