---
name: "gemini-fetch"
description: Use Gemini CLI as a web-fetch fallback when Claude WebFetch is blocked (Reddit, paywalls, flaky hosts) or when Google-grounded search is needed. Use when the user says "gemini fetch", "fetch via gemini", "WebFetch blocked", "reddit via gemini", or a skill needs a blocked URL.
---

# gemini-fetch

Use this skill when the user asks to run the migrated source command `gemini-fetch`.

## Command Template


# /gemini-fetch &lt;url-or-query&gt;

_Created: 01-08-2026 · Last updated: 26-09-2026 (H5503: mechanical route ported)_

## Default path (seconds — run this first)

The dispatch/capture bulk is ported to a deterministic script (Uprava `tools/gemini_fetch.py`, H5503). Run it first; read its report before any judgment:

```sh
# exact command it WOULD run (DEFAULT, zero network):
python3 ~/Documents/GitHub/Uprava/tools/gemini_fetch.py --url https://EXAMPLE
# real fetch — explicit --run, stdout captured to a file, bytes + exit status reported:
python3 ~/Documents/GitHub/Uprava/tools/gemini_fetch.py --url https://EXAMPLE --run --out /tmp/fetch.md
```

LLM / judgment residue only after the script report exists (H5503).

Gate that stays with the LLM: prompt-response shaping — writing the extraction prompt for the ask, judging whether the returned markdown actually answers it, and citing the Gemini provenance in the reply. The script only dispatches, captures, and masks; it cannot tell a good answer from a plausible one.

## Prerequisites

```text
npm install -g @google/gemini-cli
gemini   # first run: Sign in with Google OR set GEMINI_API_KEY
```

Auth options: Google OAuth (free tier) or `GEMINI_API_KEY` from https://aistudio.google.com/apikey

## Default path

Headless prompt (preferred for agent use):

```text
gemini -p "Fetch and summarize the main content of URL_OR_TOPIC. Quote key claims with source. If the page is blocked, try alternate public mirrors. Output markdown only." --output-format text
```

For a specific URL:

```text
gemini -p "Open and extract the full readable text of https://EXAMPLE (or explain the block). Return markdown: title, authors/date if any, body summary + 5 key quotes." 
```

JSON for parsing:

```text
gemini -p "..." --output-format json
```

## When to use

| Situation | Prefer |
|---|---|
| Normal public docs / GitHub | Claude `WebFetch` / `open_page` |
| Reddit / hard-blocked hosts | **This skill** (or Playwright DDG-hop if installed) |
| Need Google-grounded current search | **This skill** |
| Sensitive private pages | Cmd+A paste by human — never send secrets to Gemini |

## Agent rules

1. Check `gemini --version` (or `where.exe gemini`) once; if missing, install + stop with auth instructions.
2. Never pass API keys, `.env`, or credentials into the prompt.
3. Cap one call per user turn unless the user asked for multi-URL research; sleep between retries on 429.
4. Prefer Claude tools first; escalate to Gemini only on block or explicit request.
5. Cite that the extract came via Gemini CLI in the reply (provenance).

## Non-goals

- Not a general coding agent replacement for Claude.
- Not for csl-orig or money-contour file edits.

_Dr. Mārcis Gasūns_
