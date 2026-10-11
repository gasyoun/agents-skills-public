---
name: "semantic-memory"
description: 'Semantic cross-session memory for the org - search via Uprava hybrid memory_recall then lexical hub_grep fallback; durable facts as RAG-indexed Markdown memory notes. Triggers: "remember this", "what do we know about X".'
---

# semantic-memory

Use this skill when the user asks to run the migrated source command `semantic-memory`.

## Command Template


# /semantic-memory [search|store|seed-hubs|status] <query-or-fact>

_Created: 24-07-2026 · Last updated: 28-08-2026_

**Why this exists.** File `memory/` loads by session start directory (org store is invisible when
`cd`'ed into one repo). Lexical `hub_grep` finds keywords, not paraphrases. The **house hybrid
RAG** ([`recall_index.py`](https://github.com/gasyoun/Uprava/blob/main/tools/recall_index.py) /
MCP `memory_recall`, H406 BM25 + H407 MiniLM+RRF, plus the opt-in **fusion arm** H-post-bench:
RRF over BM25 + two vector stores) ranks the real hub/FINDINGS/memory-note corpus.
Marketplace **Basic Memory** remains an optional **markdown-corpus**
upgrade for vault-scale docs; it is **not** required for day-to-day fact memory.
**Do not install Memora** (NO-adopt, dual-track plan 08-08-2026).
**GrayMatter layer retired 26-08-2026 (MG ruled strike):** short-fact MCP was configured
nowhere on this host and stayed empty; its role is absorbed by memory notes, which the RAG
indexes anyway. Audit of alternatives (mem0/Letta/cognee) vs the three gates:
[docs/AUDIT_SHORT_FACT_MEMORY_LAYERS_26-08-2026.md](https://github.com/gasyoun/Uprava/blob/main/docs/AUDIT_SHORT_FACT_MEMORY_LAYERS_26-08-2026.md).

## Mode: `search` (default) — ordered retrieval

**Always this order** ([H2447](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H2447-Grok_Uprava_hub-rag-default-wire-memory-recall_08.08.26.md) — Grok 4.5, hub-RAG default wire to memory_recall; report layers separately. Never invent a memory).

1. **`memory_recall` first (hybrid/fusion hub RAG).** Prefer the MCP tool from server `uprava-recall`
   (`memory_recall`; default strategy `hybrid`, `k` 5–8; pass `strategy="fusion"` when the
   secondary store exists and latency budget allows). If MCP is missing/timed out, CLI:
   ```sh
   python ~/Documents/GitHub/Uprava/tools/recall_index.py --strategy hybrid -k 5 "<query>"
   ```
   Install + harness inventory: [RECALL_MCP_INSTALL.md](https://github.com/gasyoun/Uprava/blob/main/tools/RECALL_MCP_INSTALL.md).
2. **Lexical hub fallback** if layer 1 is thin/empty or you need exact keyword rows:
   `python ~/Documents/GitHub/Uprava/tools/hub_grep.py "<keywords>"`.

Label hits as **recall (hybrid)** · **lexical hub** in the reply.

## Mode: `store`

Store a **durable, non-obvious** fact (gotcha, ruling, danger, ownership, measured caveat):

1. Write it as a **Markdown memory note** in the owning repo's
   `~/.claude/projects/<repo-slug>/memory/<descriptive-slug>.md` (frontmatter
   `name:` + one-line `description:`; body = the fact, date-stamped, linked to
   FINDINGS §/handoff). The note joins the corpus on the next index build and is
   semantically searchable by `memory_recall` from ANY project.
2. If the fact is **destructive-risk**, also follow the global Memory-routing rule +
   `/findings-append` or `/danger-memory`.
3. One atomic fact per note; do not dump whole documents.

## Mode: `seed-hubs` (periodic / first-time)

One-shot or weekly bootstrap so semantic search has load-bearing org facts:

1. Read (do not paraphrase blindly) the load-bearing lines from:
   - [Uprava/MEGABOOK.md](https://github.com/gasyoun/Uprava/blob/main/MEGABOOK.md) theses that name sources of truth
   - Recent **🔴/🟠** rows in [Uprava/FINDINGS.md](https://github.com/gasyoun/Uprava/blob/main/FINDINGS.md) and SanskritLexicography FINDINGS
   - Open `@DECIDE` / ⭐ rows in [GTD_NEXT_ACTIONS.md](https://github.com/gasyoun/Uprava/blob/main/GTD_NEXT_ACTIONS.md)
   - SHARED_CODE one-canonical-source rulings relevant to the current work
2. Write each as a **one-sentence memory note** per the `store` mode above,
   with source pointer (full blob URL or path).
3. Cap at ~30 facts per seed pass to avoid noise. Prefer danger + ownership + "don't rebuild" facts.

## Mode: `status`

Report: (1) whether `uprava-recall` / `memory_recall` is available (MCP or CLI path),
(2) whether the fusion secondary store exists (`rv.fusion_ready()` / `$RECALL_FUSION_DB`),
(3) last successful search hit count if any,
(4) whether optional Basic Memory is installed (`pip show basic-memory`).

## Optional upgrade — Basic Memory (markdown corpus)

**When:** hub FINDINGS/manuals need **paraphrase search** at document scale beyond house
`memory_recall`, not just short facts.

```text
pip install basic-memory
# then point it at a Markdown project root (e.g. Uprava/ or ORS-FAQ Obsidian vault)
# wire as MCP per upstream docs — third-party untrusted code: read source first
```

Do **not** replace house `memory_recall` with Basic Memory for everyday search. Layers:

| Layer | Owns | Tool |
|---|---|---|
| Hub / FINDINGS / memory-note RAG (default first) | hybrid or fusion BM25+vector RRF | `memory_recall` / `recall_index.py` |
| Durable facts / rulings / danger | Markdown memory notes (`store` mode above) | `/danger-memory`, `/findings-append`, repo `memory/` |
| Lexical hub scan | `hub_grep.py` | `/context`, `/prior-art` |
| Long Markdown corpus semantic index | Basic Memory (optional) | upstream MCP after install |
| Per-directory file memory | `~/.claude/projects/.../memory/` | existing harness |

## Standing triggers (use without being asked)

- After `/findings-append` of a non-obvious gotcha → `store` a matching memory note
- After MG `@DECIDE` ruling lands → `store` the ruling sentence
- At task start when the topic is unfamiliar → `search` before rebuilding
- When two hubs contradict → escalate to CONTRADICTIONS registry

## Hard rules

- Never store secrets, API keys, personal data of students, or payment details.
- Never treat empty search as "fact does not exist" — only as "not indexed yet".
- Model attribution when writing durable docs about this skill: the executing tier+version.

_Dr. Mārcis Gasūns_
