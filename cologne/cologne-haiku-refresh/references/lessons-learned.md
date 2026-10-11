
- **The four-renderer omission.** First run only invoked `render_heatmap.py`, `render_treemap.py`, `render_sankey.py` — missing `render_cross_dict.py`. Phase 2 now explicitly names all four.
- **Locale default trap.** Renderers default to `--locale en`. Skipping `--locale ru` silently leaves ru figures at a prior commit. Phase 2 runs both.
- **Footer regex too strict.** A regex requiring `Source: CDSL <short>.txt` fails on (a) the cross-dict format (no `<short>.txt`) and (b) the Russian footer (`Источник:`, `сборка`). Phase 3's regex is now locale-agnostic.
- **`Source:` SHA-staleness is not a warning.** Figures rendered at commit X show build-X in their footer; any later commit makes them "stale" without anything being wrong. Stale SHA → INFO, not ⚠️.
- **Manifest-counter bug.** Grepping `^[-*] ` over `SUPPLEMENT_MANIFEST.md` counts the four bold section summaries (`- **(root)/** — 9 files`, etc.) as if they were file entries. Phase 7 now parses the manifest's own `**N files**` header.
- **"1 failure" without root cause is useless.** Phase 9 must always `gh run view --log-failed` for any failing run and name the actual error. (Example: 2026-05-27 MWS Pages failure was a Liquid syntax error in `DATA_DICTIONARY.md`, completely unrelated to microanalysis.)
- **Counting URLs ≠ auditing URLs.** Phase 10 must actually HEAD each unique URL with a 5s timeout. The link-rot log gets committed as audit evidence.
- **The ZIP is gitignored.** Don't hyperlink `*.zip` paths in the issue body — they 404 on GitHub. Mention the path as bare text with a rebuild instruction.

