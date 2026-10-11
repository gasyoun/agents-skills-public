---
name: deeppapernote
description: 'Generate a high-quality deep-reading note for a single paper into an Obsidian-style vault. Use when given title, DOI, URL, arXiv ID, Zotero item, or local PDF and wants a polished Markdown note.'

---

_Created: 16-07-2026 · Last updated: 29-08-2026_


# DeepPaperNote

## Localization (org vendor — read first)

This is the Sanskrit-lexicon org's localized copy of an upstream Chinese skill
([917Dhj/DeepPaperNote](https://github.com/917Dhj/DeepPaperNote)), adopted via the skill-fit audit (H999).

- **Output language:** write the note in **English by default**, or **Russian** when the paper or the
  user's request is Russian. **Never Chinese.** Section headers come from `assets/note_template.md` (English).
- **Section-name map** — the `references/*` guidance and lint rules still use the upstream Chinese section
  names; treat each as its English equivalent below and always emit the English header:

  | upstream (zh) | emit (en) | | upstream (zh) | emit (en) |
  |---|---|---|---|---|
  | 核心信息 | Key info | | 关键结果 | Key results |
  | 原文摘要翻译 | Abstract (translated) | | 深度分析 | Deep analysis |
  | 创新点 | Contributions | | 局限 | Limitations |
  | 一句话总结 | One-sentence summary | | 我的笔记 | My notes |
  | 研究问题 | Research question | | 引用 | References |
  | 数据与任务定义 | Data & task definition | | 方法主线 | Method |
  | 机制流程 | Mechanism / pipeline | | | |

  So a rule like "`原文摘要翻译` before `一句话总结`" means "Abstract (translated) before One-sentence summary".
  Any Chinese status phrase in the workflow (笔记已完成 / 已生成草稿 / 已保存到 Obsidian …) → say the English
  equivalent ("note complete" / "draft generated" / "saved to Obsidian").
- **Figure-callout fields** (from `references/figure-placement.md`): emit `> [!figure]` callouts with English
  field names — 建议位置 → **suggested position**, 放置原因 → **placement reason**, 当前状态 → **current status**;
  the catch-all `剩余图表占位` → **Remaining figures**.

Use this skill when the user wants one outcome:
- read one paper carefully
- generate a high-quality Markdown note
- save the note into an Obsidian-style vault when configured, or into the current workspace when no vault is configured

Trigger examples (EN / RU):
- `deep-read this paper and make a note`
- `generate a deep reading note for <arXiv id | DOI | PDF>`
- `сделай глубокий конспект этой статьи в Obsidian`
- `прочитай статью и создай md-заметку`

Upstream Chinese triggers still fire: `给这篇论文生成深度笔记` · `写一篇高质量论文精读笔记`.

This skill is intentionally narrow:
- it handles one paper at a time
- it does not update daily reading lists
- it does not treat a shallow abstract rewrite as a successful output
- it does not split the public entrypoint into separate setup, troubleshooting, or start commands

## Core Standard

The finished note must be more than a summary. It should reconstruct the paper's argument:
- what problem it solves
- how the task is defined
- what data or materials it uses
- how the method or analysis actually works
- what results matter most
- what the paper does not prove
- why the paper is worth keeping

Default writer persona:
- a top-tier researcher or algorithm engineer
- writing a replication-oriented lab note
- not writing a popular-science explanation
- assuming the reader can follow Python, PyTorch, training loops, and evaluation logic

The note must adapt to the paper type. Use the same base structure, but shift emphasis for AI methods, benchmarks, clinical studies, and humanities or social-science papers.

## Workflow

Follow this order:
1. resolve the paper identity
2. collect metadata
3. acquire the best available PDF
4. extract canonical raw source text: `*_raw_sections.jsonl`, `*_source_manifest.json`, and optional derived `*_full_text.md`
5. extract structural indexes and PDF assets
6. plan figure placement
7. build the full figure/table decision table
8. build the manifest synthesis bundle
9. have the model read the bundle plus raw sections and plan the note
10. run grounding lint on the note plan before drafting from it
11. have the model write the note
12. lint the final note — if the lint output contains `passes_style_gate: false`, apply the Style Gate Enforcement rule before advancing to step 13, 14, or 15
13. perform `final_quality_review` after lint passes
14. perform `final_readability_review` after the quality review passes
15. write into Obsidian

This is the required workflow for a normal single-paper note request, not a loose suggestion.
Unless this skill explicitly marks a stage as optional, required stages must not be silently skipped, reordered into a shortcut, or treated as complete just because a partial artifact already exists.

Global no-short-circuit rule:
- do not stop after only the early stages and present the workflow as finished
- do not treat slowness, inconvenience, or temporary uncertainty as permission to bypass a required stage
- do not replace the declared workflow with an improvised shortcut
- if a required stage fails, only do one of three things:
  - retry that stage
  - enter a fallback that is explicitly allowed by this skill
  - stop and report which stage is blocked and which downstream required stages remain incomplete
- do not describe the whole task as complete while required downstream stages are still pending

Completion-language rule:
- say `note complete` only when the required workflow is actually complete
- say `draft generated` when drafting is done but lint, final readability review, or save is still pending
- say `passed lint` only when lint has actually been run and passed
- say `saved to Obsidian` only when the write step has actually succeeded
- do not treat `lint passed` as equivalent to `the whole note is fully polished`
- if final readability review is still pending, explicitly say the draft passed script lint but has not finished final language review
- if the workflow stopped early, name the current stage and the still-missing required stages instead of using completion language
- lint is a floor, not the writing objective

## Core Execution Contract

`SKILL.md` plus the generated `synthesis_bundle.json` must be enough to complete a normal note-generation run.
Files under `references/` are optional stage-specific deep dives, not a default reading checklist.

Non-negotiable rules:
- evidence-first: draft from the synthesis bundle, `source_manifest`, raw sections, coverage metadata, explicit `note_plan`, and inspected paper evidence; never finish from title/abstract/headings alone
- raw-source authority: for ordinary PDFs, `*_raw_sections.jsonl` and `*_source_manifest.json` are the canonical reading material; old top-N evidence buckets, truncated `section_texts`, and `candidate_chunks` are not model-facing writing inputs
- fail-closed: if a usable PDF or sufficient evidence cannot be obtained after supported acquisition paths, stop and ask for better source material rather than producing a finished degraded note
- model-first: scripts structure evidence, but the model must decide emphasis, contribution, mechanism, limitations, and final Chinese prose
- explicit planning: before drafting, save a compact JSON `note_plan` such as `<note>.plan.json` or `*_note_plan.json`; pass it to `scripts/lint_note.py --plan-file ...`
- grounding gate: after the JSON `note_plan` exists, run `scripts/lint_grounding.py --note-plan ... --source-manifest ... --bundle-json ... --figure-decisions ...`; each substantive section must cite valid `section_id` values or valid page ranges
- required structure: include the canonical required sections, with `原文摘要翻译` before `一句话总结` and a dedicated `创新点` section immediately after `原文摘要翻译`
- abstract translation: when abstract metadata exists, `原文摘要翻译` is a faithful Chinese translation of the original abstract, not a bilingual block and not the model's own summary
- mechanism depth: method, framework, and system papers should include `### 机制流程` under `方法主线`, normally as a 3 to 4 step numbered flow with input, operation, and output destination
- placeholder-first figures: plan major figure/table placeholders first; replace one only when identity match and visual usability are both strong; otherwise keep the placeholder
- final quality gates: lint is a floor; after lint passes, first run `final_quality_review` for analytical depth, then run `final_readability_review` for language polish, and rerun lint if either review edits the note
- Obsidian-first save: if a vault is configured, treat it as the required target, create the paper-local `images/` directory, and never present a fallback/workspace write as a successful vault save

Reference usage policy:
- do not load every reference file by default
- consult `references/workflow.md` only for detailed data contracts or pipeline debugging
- consult `references/evidence-first.md`, `references/deep-analysis.md`, or `references/final-writing.md` only when the paper is complex or the draft is too shallow
- consult `references/figure-placement.md` only for ambiguous figure/table placement or image replacement decisions
- consult `references/obsidian-format.md` only for Markdown, vault, frontmatter, or reference-link formatting details
- consult `references/note-quality.md` or `references/paper-types.md` only for final review or domain adaptation
- consult `references/metadata-sources.md` only when metadata is incomplete, and `references/architecture.md` only for repository maintenance decisions

## Tool and Source Priority

Prefer the strongest available source in this order:
1. local PDF path given by the user
2. local Zotero item and local Zotero attachment if available
3. DOI and publisher metadata
4. arXiv or open-access PDF sources
5. Semantic Scholar or OpenAlex for metadata backfill

Before resolving the paper, actively check Zotero integration: attempt to call the Zotero MCP tool (for example, search for the paper title or list libraries). If the tool responds without error, Zotero is available and the local-library-first rule below applies. If the call fails or the tool is not present, record "Zotero not available" and proceed without it. Do not skip this check — the check itself determines whether local-library-first applies.

Local-library-first rule (applies only when the Zotero check above succeeds):
- search the local Zotero library first using the paper title, DOI, or arXiv id
- If Zotero finds the paper, treat that result as the canonical identity resolution step.
- If the attachment path is not exposed by the integration, use `scripts/locate_zotero_attachment.py` with the attachment key and filename to find the local PDF under the user's Zotero storage.
- If a local attachment path is available, pass it forward as the preferred PDF source.
- If no local attachment is found, still use the library-resolved metadata to avoid title ambiguity, then fall back to network PDF acquisition only for the file itself.
- Do not let a weaker title-only internet match override a confident local-library hit.

## Limits

- If the paper identity is ambiguous, confirm before writing.
- If the PDF is unavailable after all supported acquisition paths have been tried, stop and report what input is needed; do not produce a degraded, provisional, or abstract-only note as the finished output. Supported acquisition paths include local PDF, Zotero attachment, metadata `pdf_url`, direct PDF URL, arXiv/open-access sources, publisher PDF if accessible, DOI enrichment, and any other current fetch path implemented by the workflow.
- Placeholder-first figure planning is required; image extraction is optional and must never reduce textual coverage.

_Dr. Mārcis Gasūns_
## References

Content split (H5588) — sections below were moved verbatim, read on demand:

- [references/output-and-model-rules.md](references/output-and-model-rules.md) — Output rules (save targets, note structure, figure/table placeholder and insertion rules, style gate) and the model-first rule block
- [references/scripts-and-pipeline.md](references/scripts-and-pipeline.md) — Bundled script catalog, preferred usage pattern, Python interpreter rule, troubleshooting and pipeline status
