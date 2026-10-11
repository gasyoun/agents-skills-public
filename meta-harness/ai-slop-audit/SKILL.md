---
name: "ai-slop-audit"
description: GPTZero-style AI-slop investigation — external AI detectors (% AI / Mixed / Human), vibe-citation hallucination check, and claim-vs-source verification. Use for "is this AI-generated", "60% Mixed", GPTZero-like audit, hallucinated references, or consulting-report probes.
---

# ai-slop-audit

Use this skill when the user asks to run the migrated source command `ai-slop-audit`.

## Command Template


# /ai-slop-audit [--mode detect|citations|full] [--detector gptzero|winston|originality|sapling|manual] path-or-URL-or-paste

_Created: 06-08-2026 · Last updated: 05-09-2026_

Reproduces the **three-layer** workflow used in GPTZero-style investigations
([PwC Middle East series](https://gptzero.me/news/investigations-pwc/)):

1. **AI detector score** — document-level HUMAN / MIXED / AI + % / confidence — never invent a %
2. **Hallucination / vibe-citation check** — OK / FLAWED / FAKE / UNSURE
3. **Claim × source support** — CONFIRMED / NOT SUPPORTED / CONTRADICTED / UNVERIFIABLE

Plus optional `/litredaktor`-style marker flags (read-only).

This command **does not invent** a home-grown probabilistic detector. Scores come from
**external detectors** (API or manual paste-into-UI). Local qualitative layers fill the gaps
when no API key is present.

Detector catalog: [`~/.claude/references/ai-slop-detectors.md`](https://github.com/gasyoun/claude-config/blob/main/references/ai-slop-detectors.md).
Helper: [`~/.claude/scripts/gptzero_detect.py`](https://github.com/gasyoun/claude-config/blob/main/scripts/gptzero_detect.py).

## Modes

| Flag | What runs |
|------|-----------|
| `--mode detect` | External detector only → score table |
| `--mode citations` | Bibliography extract → vibe-citation verdicts |
| `--mode full` **(default)** | Detector + citations + claim audit + style markers |

## Inputs

Accept any one of:

- Path to `.md` / `.txt` / `.pdf` / `.docx` (extract text first)
- URL of a public report/PDF (fetch, or instruct the human to download)
- Pasted prose in the chat
- Directory of extracts (process the largest prose file, or ask which)

**Privacy:** never commit raw scanned third-party proprietary text to a public repo. Park
results under a private hub path or the owning worktree only.

## Keys (env only)

`GPTZERO_API_KEY` · `WINSTON_API_KEY` · `ORIGINALITY_API_KEY` · `SAPLING_API_KEY` ·
`HF_TOKEN` (only when running HF inference models). Never store a key in this file, in a
commit, or in chat logs.

```powershell
$env:GPTZERO_API_KEY = "..."
$env:WINSTON_API_KEY = "..."
$env:ORIGINALITY_API_KEY = "..."
```

## Phase 0 — Preflight

1. Resolve text (file / URL / paste). Image-only PDF ⇒ OCR first, or stop with a clear blocker.
2. Count words. Detectors are unreliable under ~**150–250 words**; warn, and still proceed if
   the human insists.
3. Choose a detector:

| Preference | When |
|------------|------|
| `gptzero` | Default if `GPTZERO_API_KEY` is set — matches the investigation labels (HUMAN_ONLY / MIXED / AI_ONLY) |
| `winston` | If `WINSTON_API_KEY` / Bearer token is set |
| `originality` | If `ORIGINALITY_API_KEY` is set (content teams) |
| `sapling` | Light free-tier style checks when `SAPLING_API_KEY` is set |
| `manual` | No keys — open the detector web UI recipe + qualitative layers only |

4. Never fabricate a % score. If no detector ran, write
   **`Score: UNAVAILABLE (no API key / manual pending)`**.

## Phase 1 — External AI detector

### 1a. GPTZero (preferred for label parity)

Docs: [developers](https://gptzero.me/developers) · [API](https://gptzero.stoplight.io/).
Endpoint `POST https://api.gptzero.me/v2/predict/text`, header `x-api-key: $GPTZERO_API_KEY`.

```bash
# Prefer the bundled helper:
python ~/.claude/scripts/gptzero_detect.py path/to/text.txt

# Or curl:
curl -sS -X POST "https://api.gptzero.me/v2/predict/text" \
  -H "Accept: application/json" \
  -H "Content-Type: application/json" \
  -H "x-api-key: $GPTZERO_API_KEY" \
  -d '{"document": "<escaped text>"}'
```

**Map response fields** (names may evolve — prefer the live docs):

| Field | Report as |
|-------|-----------|
| `documents[0].document_classification` | `HUMAN_ONLY` / `MIXED` / `AI_ONLY` |
| `class_probabilities` / AI probability | **~N% AI-generated or Mixed** (state which class) |
| `confidence_category` | high / medium / low |
| sentence / highlight fields | top AI-flagged sentences (cap 8 in the report) |

GPTZero's Hallucination Check is a **separate product** — if the human has dashboard access,
link the scan URL; never invent its results.

### 1b. Winston AI

Docs: [docs.gowinston.ai](https://docs.gowinston.ai/) ·
`POST https://api.gowinston.ai/v2/ai-content-detection`, header
`Authorization: Bearer $WINSTON_API_KEY`. Report the **Human Score** and any sentence-level
breakdown, normalized to one label: high human → HUMAN-leaning; mid → MIXED; low human →
AI-leaning.

### 1c. Originality.ai

Docs: [docs.originality.ai](https://docs.originality.ai/api-v2-0-new) — scan/ai endpoint family
with `X-OAI-API-KEY`. Report Originality's AI % and their plagiarism score **separately**;
never conflate the two.

### 1d. Sapling / Copyleaks / Grammarly / Hive / Pangram

Use only when the human names them or a key is present. Links and caveats:
[`references/ai-slop-detectors.md`](https://github.com/gasyoun/claude-config/blob/main/references/ai-slop-detectors.md).

### 1e. Open-source / local (optional, never default)

| Tool | Notes |
|------|--------|
| [Binoculars](https://huggingface.co/blog/dmicz/binoculars-text-detection) | Zero-shot; needs a GPU + two models; lags the newest LLMs |
| [Fast-DetectGPT](https://github.com/baoguangsheng/fast-detect-gpt) | Research detector |
| `openai-community/roberta-base-openai-detector` (HF) | **GPT-2-era** — never proof about a current LLM |
| Open Pangram / EditLens | Research-grade; check the licence before bundling |

If one is run, label the result **`local/research — not calibrated like GPTZero`**.

### 1f. Manual UI fallback (no key)

Ask the human to paste into **one** of [gptzero.me](https://gptzero.me/) ·
[gowinston.ai](https://gowinston.ai/) · [originality.ai](https://originality.ai/), then paste
the numbers back. While waiting, run Phases 2–4 so the session still produces value.

## Phase 2 — Style / "AI slop" markers (qualitative)

Run the **inverse** of `/litredaktor` as a **read-only** audit (do not rewrite unless asked).
Flag the density of:

- Fake honesty / "It's not X — it's Y"
- Robot-accountant bullet sheets
- Vague product definitions ("dynamic mechanism… powered by…") with no operational detail
- Metadata-title citations (`Title | Site`) and `utm_source=chatgpt.com` URLs
- Repeated boilerplate paragraphs / duplicated subtitles
- Grand claims on thin evidence

Emit **marker count + 3–8 short quotes**, not a rewrite.

## Phase 3 — Vibe-citation / hallucination check

Extract the bibliography plus every in-body footnote/URL. For each citation (cap **40**;
stratified sample if more):

| Verdict | Meaning (aligned with GPTZero's "vibe citation" spirit) |
|---------|--------------------------------------------------------|
| **OK** | Source exists; title/authors/year roughly match |
| **FLAWED** | Real source, wrong year / missing authors / dead URL / wrong journal — plausibly human error |
| **FAKE / VIBE** | Fabricated or hybrid (wrong authors + wrong title, DOI resolving to nothing, a title that never existed) |
| **UNSURE** | Paywalled / ambiguous match / condensed consulting style |

Method: parse author, title, year, DOI, URL → web-search the exact title + DOI and open the
best hit → compare components (a missing letter is human error, not FAKE) → flag
`utm_source=chatgpt.com` and metadata-pipe titles as **AI-process signals**, not automatic FAKE.

Table `| # | Citation (short) | Verdict | Evidence URL | Notes |`, then
`OK=a · FLAWED=b · FAKE=c · UNSURE=d`.

Sibling: `/bib-hygiene` covers `.bib` mechanical hygiene only — this phase is **existence +
fidelity**, not key-dedup.

## Phase 4 — Claim × source support

Same contract as `/verify-claims`, scoped to **cited body claims**:

| Status | Meaning |
|--------|---------|
| **CONFIRMED** | The cited source supports the claim as stated |
| **NOT SUPPORTED** | The source exists but does not contain the claim/stat |
| **CONTRADICTED** | The source says the opposite / a different number |
| **UNVERIFIABLE** | No accessible primary |

Cap ~25 load-bearing claims (numbers, product adoptions, "country X uses Y"). Ignore pure
marketing fluff unless it names a product + a customer.

## Phase 5 — Report (required shape)

Write a durable `.md` when the audit is recall-worthy (private repo, `RESULTS_LOG.md`, or the
investigation folder). Chat-only only on an explicit `CHAT-ONLY-OK`.

```markdown
# AI-slop audit — <title or file>

_Created: DD-MM-YYYY · Last updated: DD-MM-YYYY_
_Detector: GPTZero API | Winston | … | UNAVAILABLE_
_Model running audit: <tier + exact version>_

## Executive summary

| Field | Value |
|-------|--------|
| Classification | HUMAN_ONLY / MIXED / AI_ONLY / UNAVAILABLE |
| Score line | e.g. **60% AI-generated or Mixed** (quote the vendor wording) |
| Confidence | high / medium / low / n/a |
| Citations | OK / FLAWED / FAKE / UNSURE counts |
| Claims | CONFIRMED / NOT SUPPORTED / CONTRADICTED / UNVERIFIABLE |
| Overall diagnosis | 1–2 sentences, evidence-based, no moralizing |

## 1. Detector — vendor, endpoint/UI, word count, date; classification + probabilities; top AI-highlighted sentences (≤8)
## 2. Style markers — density + example quotes
## 3. Citations — | # | Citation | Verdict | Evidence | Notes |
## 4. Claims vs sources — | # | Claim | Status | Cited as | Evidence | Notes |
## 5. Caveats — detectors err (false positives on ESL / formal / short text); a score is not proof of misconduct; do not republish third-party PDFs.
```

End the chat reply with the same executive-summary table (compact) plus the path to the
durable file.

## Ethics & non-goals

- **Not a final verdict** on authorship, employment, or academic integrity.
- **Not** `ai-assessment-scale` (AIAS levels for *software project disclosure*).
- **Not** `/paper-referee` (the full scholarly Major/Minor pass) — escalate there for Axx papers.
- **Not** a `/litredaktor` rewrite — only flag, unless asked to clean.
- Never scrape a detector UI against its ToS; use official APIs or human paste-back.

## Compose with siblings

| Need | Skill |
|------|--------|
| Claims only | `/verify-claims` |
| De-AI rewrite after the audit | `/litredaktor` |
| `.bib` mechanical fix | `/bib-hygiene` |
| Hostile paper review | `/paper-referee` |
| Project AI-usage disclosure | `ai-assessment-scale` |

## Quick start

```
/ai-slop-audit path/to/report.pdf
/ai-slop-audit --mode detect --detector gptzero ./chapter.md
/ai-slop-audit --mode citations https://example.com/report.pdf
```

This file is the canonical source; the Codex and Grok twins are generated by
`scripts/generate_playbook_twins.py`.

_Dr. Mārcis Gasūns_
