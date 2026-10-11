---
name: "chat-knowledge-audit"
description: 'Session-store knowledge census loop: extract chat knowledge from all four harness stores (Claude/Codex/Grok/OpenCode) via tools/chat_knowledge_extract.py, read-only — deterministic cross-match against the hub corpus — dated report — then adjudicate the un-hubbed residue in CAPPED batches (GENUINE lands as hub rows in the same batch, CHATTER recorded); security findings route separately (marker row, no rotation without a human). OxAlpha (paid, bounded) / Sonnet 5 (claude-sonnet-5). Triggers: "chat knowledge audit", "what lives only in chat", "adjudicate the candidates".'
---

# chat-knowledge-audit

Use this skill when the user asks to run the migrated source command `chat-knowledge-audit`.

## Command Template


# /chat-knowledge-audit [--census | --adjudicate [CANDIDATES-JSON] | --stage B]

_Created: 29-08-2026 · Last updated: 05-09-2026_ (C44)

Encodes the session-store knowledge census loop of [H3598](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H3598-OxAlpha_Uprava_chat-only-knowledge-audit_27.08.26.md) (stage-A census: 1 522 sessions, 80.9 % hub-anchored, 136 genuine chat-only candidates) and [H3613](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H3613-OxAlpha_Uprava_h3598-candidates-adjudication_28.08.26.md) (bounded adjudication of that residue) as one repeatable skill — census card **C44** (skill-mine sheet H3556–H3614, MG chat vote 28-08-2026). Tool: [`Uprava/tools/chat_knowledge_extract.py`](https://github.com/gasyoun/Uprava/blob/main/tools/chat_knowledge_extract.py) (exists — consume, never rebuild); programme record: [REPORT_CHAT_ONLY_KNOWLEDGE_AUDIT_27-08-2026.md](https://github.com/gasyoun/Uprava/blob/main/docs/REPORT_CHAT_ONLY_KNOWLEDGE_AUDIT_27-08-2026.md).

The principle: **chat is a transport, hubs are the product — and the stores that hold the chat are secret surfaces.** The census classifies and never writes into a store; the adjudication lands hub rows in the same batch it reads (a verdict without its row is the defect); anything credential-shaped leaves the candidates lane immediately and takes the security route, where the standing ruling is *mark, don't rotate*.

Triggers: "chat knowledge audit", "what knowledge lives only in chat", "adjudicate the chat-only candidates", "re-run the session census", a `/skill-mine` or `/weekly-review` pass that finds session knowledge with no hub anchor.

## Stage 0 — Fences (read before any run)

1. **Stores are read-only, always.** Claude `~/.claude/projects/`, Codex `~/.codex/sessions/`, Grok `~/.grok/sessions/`, OpenCode `opencode.db` (sqlite **strictly `mode=ro`**). Any write to a session store = named FAIL.
2. **Stores are credential-bearing** ([DANGER_FACTS row](https://github.com/gasyoun/Uprava/blob/main/DANGER_FACTS.md), H3598: `sk-`-shaped keys in 50 sessions / 17 distinct values — floors, not totals). The raw extract NDJSON inherits that: it is gitignored by design, written to a local evidence dir or scratchpad, **never committed, never synced, never pasted**. Committing a raw extract = named FAIL.
3. **Stage B (full Grok text, ~7.7 GB / 45 913 files) is cost-gated.** OxAlpha is a paid lane since 26-08-2026 (FINDINGS §568); stage B runs only when a human names it — `--stage B` without that go is a refusal, not a default.
4. **No rotation without a human.** MG ruled 28-08-2026: exposed-key risk stays *marked* (the DANGER_FACTS row IS the marker), not rotated. The skill never rotates, revokes, or purges; it updates the marker row and stops (see Stage 5).
5. Prior art: `python Uprava/tools/hub_grep.py "chat knowledge census"` — the previous dated report and candidates JSON are the baseline; a census that ignores its predecessor re-litigates 1 522 verdicts.

## Stage 1 — Extract (census lane)

```
python Uprava/tools/chat_knowledge_extract.py extract --stage A --out <local-evidence>/extract-stage-a.ndjson
```

Stage A = Claude + Codex + OpenCode text, Grok **metadata only** (H-ids/cwd/dates). Extraction contract (already in the tool — stated here so nobody "improves" it away): user prompts + final assistant text only; tool bodies, hook noise, environment context, permissions preamble skipped; caps 4 000 chars/message, 60 messages and 60 000 chars/session. The caps mean every count downstream is a **floor** — report them as floors.

## Stage 2 — Deterministic cross-match

```
python Uprava/tools/chat_knowledge_extract.py match --extract <ndjson> --out <local-evidence>/match-extract.json
```

Token / `H###` / dated-header overlap against the hub corpus: FINDINGS.md ×2, GTD_NEXT_ACTIONS.md, ARTICLES.md, PROJECT_INTERLINKS.md, SHARED_CODE.md, DANGER_FACTS.md, the handoffs registry + files, per-repo `.ai_state.md` + CHANGELOGs (~494 docs at H3598). Mechanical only — no model judgment reads matched sessions. Denominators first: sessions per store, anchored %, residue buckets (cron noise · boilerplate · meta-only · **genuine candidates**). H3598 baseline: 1 231/1 522 anchored (80.9 %); residue 62 cron + 10 boilerplate + 83 Grok meta-only + 136 candidates.

## Stage 3 — Dated census report

`report --match <json>` renders the skeleton; land `docs/REPORT_CHAT_ONLY_KNOWLEDGE_AUDIT_DD-MM-YYYY.md` in Uprava (worktree off `origin/main`) with: per-store denominators · anchored % + **delta vs the previous census** · residue bucket table · candidates JSON committed (`docs/evidence/chat-knowledge/candidates-stage-a.json` — session ids + dates + hint tokens only, no raw text) · security tally (counts and shapes, **never values**). Register: CHANGELOG bullet + GTD @DO adjudication row for the residue. Census stops here — `--census-only` posture is the default; adjudication is its own bounded pass (a scanner that fixes while it counts cannot be re-run — the same D2 separation as `/hygiene-track`).

## Stage 4 — Adjudicate the residue (bounded batches)

Input: the committed candidates JSON (or `--adjudicate PATH`). Method, verbatim from H3613:

1. **Rebuild the local raw extract first** (`extract --stage A`) — the NDJSON is machine-local and gitignored; candidates JSON alone has no text to judge.
2. Read **capped batches of ~15 sessions**; hard cap **10 batches per pass**. The cap hits ⇒ stop with an honest partial-verdict table, never a silent tail-drop.
3. Classify each session: **GENUINE** (a real decision / analysis / design with no hub row) vs **CHATTER** (automation noise, plan-scratch, subagent scratch, superseded drafts).
4. **GENUINE lands in the same batch** — a FINDINGS entry or a GTD `@DO` row with the next physical action, grouped by theme; a verdict table row alone is not landing.
5. Anything credential- or personal-data-shaped found mid-batch leaves the lane → Stage 5. It is never a GENUINE hub row with the value in it.
6. Land `docs/REPORT_H3598_CANDIDATES_ADJUDICATION_DD-MM-YYYY.md` (per-candidate verdict table: session id · store · date · verdict · hub row link or CHATTER reason) + CHANGELOG + close. Adjudicated candidates drop out of the next census's residue — that closes the loop.

## Stage 5 — Security findings route (separate lane, always)

Key-shaped or personal-data findings **never ride the candidates report**. Route: update/extend the DANGER_FACTS marker row (counts, distinct-value floors, stores affected — no values) + `context_facts.yaml` mirror + one GTD `MG @DECIDE`/`@DO` row when the risk posture changed. The MG 28-08-2026 no-rotation ruling stands until a human reverses it: revisit only on evidence a value is live or a store left the machine. Purge/rotation acts are `/secret-purge` territory, human-gated — this skill only marks.

## Dedupe fence

- [`/inbox-triage`](https://github.com/gasyoun/claude-config/blob/main/commands/inbox-triage.md) — routes the human-authored `Inbox.md`; this skill routes what sessions said and never wrote down.
- [`/semantic-memory`](https://github.com/gasyoun/claude-config/blob/main/commands/semantic-memory.md) — per-session memory-store curation; this is an org-wide census against the hub corpus, not a memory write path.
- [`/skill-mine`](https://github.com/gasyoun/claude-config/blob/main/commands/skill-mine.md) — mines session footprints for repeated **workflow shapes**; this mines them for **un-hubbed knowledge**. Same stores, different product; a census hit that is a workflow shape gets referred there, not carded here.
- [`/hygiene-track`](https://github.com/gasyoun/claude-config/blob/main/commands/hygiene-track.md) track D / [`/secret-purge`](https://github.com/gasyoun/claude-config/blob/main/commands/secret-purge.md) — secrets in **repos** and their destructive remediation; Stage 5 here covers secrets in **session stores** and only marks.
- [`/findings-append`](https://github.com/gasyoun/claude-config/blob/main/commands/findings-append.md) — the landing vehicle Stage 4 calls per GENUINE item; it owns hub-row placement, this skill owns the loop.

Fail = any write to a session store · OpenCode opened without `mode=ro` · raw extract NDJSON committed, synced, or quoted with values · stage B run without a named human go · an adjudication batch over ~15 sessions or a pass over 10 batches · a GENUINE verdict with no hub row in the same batch · a cap-hit pass without its partial-verdict table · a key value in any report, hub row, or chat · a rotation/purge attempted from this skill.

_Dr. Mārcis Gasūns_
