## Scripts

Use these bundled scripts rather than rebuilding the workflow from scratch:
- `scripts/check_environment.py`
- `scripts/create_input_record.py`
- `scripts/locate_zotero_attachment.py`
- `scripts/resolve_paper.py`
- `scripts/run_pipeline.py`
- `scripts/collect_metadata.py`
- `scripts/fetch_pdf.py`
- `scripts/extract_source_text.py`
- `scripts/extract_evidence.py`
- `scripts/extract_pdf_assets.py`
- `scripts/plan_figures.py`
- `scripts/plan_figure_table_decisions.py`
- `scripts/build_synthesis_bundle.py`
- `scripts/lint_grounding.py`
- `scripts/lint_note.py`
- `scripts/materialize_figure_asset.py`
- `scripts/write_obsidian_note.py`

Preferred usage pattern:
1. if local bibliography integration is available, search the local Zotero library first
2. if the library resolves the paper, inspect child attachments; if needed use `scripts/locate_zotero_attachment.py` to find the local PDF
3. use `scripts/create_input_record.py` to materialize a trusted JSON input record
4. run `scripts/run_pipeline.py` on the JSON record or original exact source to produce the bundle
5. read the bundle yourself
6. write the note in your own words
7. lint the note
8. write it into Obsidian only after lint passes and the final readability review is complete

Python interpreter rule:
- DeepPaperNote requires Python `>=3.10`.
- Before running repository scripts, check the interpreter version instead of assuming the current shell default is compatible.
- If the default `python3` is below `3.10`, automatically look for another available interpreter that satisfies the requirement, such as `python3.12`, `python3.11`, `python3.10`, `/opt/anaconda3/bin/python3`, `/opt/homebrew/bin/python3`, or `/usr/local/bin/python3`.
- Use the first compatible interpreter you find and continue with that interpreter for the repository scripts in the current task.
- If no compatible interpreter is available, stop and clearly tell the user which interpreter was found, which version it reported, and that DeepPaperNote requires Python `>=3.10`.

Troubleshooting rule:
- use `scripts/check_environment.py` only when a concrete dependency or integration question is blocking execution
- explain required dependencies, optional enhancements, and downgrade behavior directly rather than redirecting the skill into a separate troubleshooting workflow
- do not feature environment inspection as a public pseudo-command surface

Current status:
- the single-paper deterministic core pipeline is implemented as an MVP
- `scripts/run_pipeline.py` now defaults to building a model-facing synthesis bundle
- `scripts/write_obsidian_note.py` can write the final note into a target vault
- patch the scripts rather than replacing the workflow ad hoc

