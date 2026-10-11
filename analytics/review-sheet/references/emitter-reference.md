- `repo-slug` — the owning repo, lowercase (`commentarystrategies`, `csl-atlas`, `sanskritlexicography`).
- `topic` — what's being judged (`sundarakanda-commentaries`, `h4-semantic-field`, `xref-shared-core`, `renou-hypotheses`).
- `scope` — the narrowing detail that disambiguates repeat rounds on the same topic (`35-37` for a sarga/song range, `batch2`, `pilot`, a date `05-07-26` if nothing else narrows it).
```sh
pip install "csl-pyutil @ git+https://github.com/sanskrit-lexicon/csl-pyutil@main"
```
```python
from csl_pyutil import render_review_sheet
html = render_review_sheet(
    items=[{"id": ..., "filt": ..., "title": ..., "badges": [...], "question": "...",
             "panels": [("heading", "html body"), ...], "note_placeholder": "..."}, ...],
    config={
        "sheet_id": "<repo-slug>-<topic>_<scope>",   # the naming convention below, without the _review.html suffix
        "title": "...", "subtitle": "...", "footer": "...",
        "approve_label": "...", "reject_label": "...",   # button text, e.g. "Accept correction" / "Keep as-is"
        "filters": [("key", "label"), ...],              # filter-bar buttons; "all" and "unvoted only" are added automatically
        "generated": "...",   # a date string passed explicitly, never computed inside the emitter, so output is reproducible
        # ------- the 19-07-2026 standard (V1–V8) — MANDATORY for every new sheet -------
        "show_ids": True,             # V3: visible copyable id chip per card — MG cites ids back
        "note_min_height_px": 88,     # V6: taller free-text note box
        "save_as": "<Repo>/review/<sheet_id>_decisions.json",  # V8: in-sheet banner binding export → destination
        # "rating": {"label": "Direct Assessment", "scale": 5, "threshold": 3, "approve_min": 4},
        #   ^ V1/V5, OPTIONAL per sheet semantics: 1–5 click buttons BELOW the card (never a slider,
        #     never above); approve auto-raises the rating to approve_min; export items gain "rating".
    },
)
```
Write the returned string to the sheet path per Phase 3 below. Only fall back to hand-writing
HTML if the package genuinely can't be installed in this environment (no network, etc.) — note
that explicitly in the report if it happens.

**The 19-07-2026 standard (V1–V8) is the ONLY *emitter UX* standard** — ruled by MG in the
[h178_da vote meta-note](https://github.com/gasyoun/SanskritLexicography/blob/master/RussianTranslation/pwg_ru/H178_DA_VOTE_ISSUE_REGISTER_2026-07-19.md)
(§2) and shipped as csl-pyutil
[v0.3.0](https://github.com/sanskrit-lexicon/csl-pyutil/releases/tag/v0.3.0).
**Content-grade is separate and mandatory** (H2046) — see Phase 0 paragraph above and
[REVIEW_SHEET_CONTENT_STANDARD_2026.md](https://github.com/gasyoun/Uprava/blob/main/docs/REVIEW_SHEET_CONTENT_STANDARD_2026.md);
emitter green does not waive C1–C8 / U1–U7. Besides the
mandatory config keys above: give every item a `title_href` (V4 — the card header links to
the full source entry; for PWG headwords use `RussianTranslation/src/review_sheet_standard.py`'s
`pwg_entry_href()`; omit only when no per-item URL exists and say so in the report), and wrap
Russian-text content with `csl_pyutil.mark_cyrillic()` after escaping (V7 — the words under
judgment are color-highlighted; never highlight German/IAST). Hand-rolled sheet HTML is banned
for NEW sheets in every repo — python repos call the emitter; JS-stack repos (csl-atlas) mirror
these features in their template until their port to the shared emitter lands.

**Contrast requirement ([v0.3.1](https://github.com/sanskrit-lexicon/csl-pyutil/releases/tag/v0.3.1)+).**
The sheet is dark-themed, so it MUST declare `<meta name="color-scheme" content="dark">` +
`:root { color-scheme: dark }` and force the note `textarea`'s dark background + light text
(`!important`, `-webkit-text-fill-color`, `::placeholder`) — otherwise a light-mode browser/OS
renders the textarea white-bg with light text (invisible letters, found 19-07-2026 mid-vote on
the H1323 sheet — H1323 (Opus 4.8) — PWG ghostword triage stages 1–4). The python emitter now bakes this in; JS-stack mirror templates (csl-atlas)
must carry the same until they port to the emitter.

The emitter produces, self-contained (inline CSS+JS, no server, no CDN — opens from disk years later):

- Header: title, date, item count, source description.
- Per item: title, badges, rendered `question` (the judgment context — text, diff, quote, links), optional `panels` (supporting detail blocks), three buttons — ✅ approve · ❌ reject · ⏸ defer — plus a free-text note field.
- Running tally (approved/rejected/deferred/unvoted) pinned on screen; keyboard shortcuts (a/r/d, arrows) for fast passes.
- Votes persist to `localStorage` keyed by sheet ID as the user clicks (survives an accidental close).
- **"Download `<sheet-name>_decisions.json`"** button (named per the convention above, not the bare word "decisions") exporting `{ sheet_id, generated, decided, items: [{id, decision, note}] }` — includes unvoted items with `decision: null` so the consumer can detect an incomplete pass.
- **Auto-save via File System Access API, with the download button kept as fallback.** On Chromium browsers (`window.showSaveFilePicker` available), a "Save to folder" control lets the user pick a save target once, then every vote change (debounced ~1s) writes straight to that handle — no repeated download clicks, no browser-default-download-folder surprises (H779, 12-07-2026: a sheet's decisions.json landed in the wrong repo's `review/` folder because that happened to be the browser's configured download directory). Falls back silently to the manual Download button when the API is unavailable (Firefox, or the user declines the picker).
- **Button-legend footer, always.** Explains the three verbs in one line each: **Approve** = accept the proposed change shown on the card (there is no separate "approve as-is" — approving means agreeing the correction should be made as written); **Reject** = keep the current entry unchanged; **Defer** = not sure yet, decide later; plus a line noting the free-text note field is for requesting a partial tweak instead of an outright reject. MG rule 12-07-2026 (H779), after asking "I approve, I approve as-is, or what?" on a sheet with no legend.
