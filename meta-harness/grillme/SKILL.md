---
name: grillme
description: 'Homework-first interrogation: exhaust the estate surfaces, then grill MG with 10–20 deep, evidence-cited questions (each with options + ONE recommendation) before any plan is written. Use for «/grillme», «grill me», «задай мне вопросы», «ask me tens of questions». Companion of /ask (which front-loads a single interview): /grillme is adversarial-depth, homework-first, run in rounds.'
---


# /grillme — homework-first interrogation rounds

_Created: 05-09-2026 · Last updated: 02-10-2026_

_At MG's ask during the adjudication-first week (H4116): «you can /grillme to find out
what would make more sense, ask me tens of questions or interlinking and data reuse,
but first do your homework»._ Canonical since 05-09-2026: this skills-dir file
(`~/.claude/skills/grillme/SKILL.md`); `~/.claude/commands/grillme.md` is a thin
pointer for the TUI. Body restored 02-10-2026 (H5708) after the H5567 refresh copied
the command's thin pointer over it, leaving a command → skill → itself loop.

**The one invariant: the agent never asks what the estate already answers.** A question the
files could resolve is the agent's defect, not MG's time.

## Phase 0 — Homework (mandatory, before the first question)

1. Exhaust the estate surfaces for the topic: `memory_recall` / `/context`; kosha
   `data/manifest/datasets.json` (121 registered datasets); Uprava `DATA_LAYERS_CENSUS.md`,
   `REUSE_INDEX.md`, `PROJECT_INTERLINKS.md` + `interlinks_edges.tsv`,
   `FEATURES_INDEX.md` (SanskritLexicography), `SHADOW_ASSETS_POINTERS.md`; sibling repos'
   `.ai_state.md`, registries, `FINDINGS.md`.
2. For every candidate question, try to answer it from the surfaces. What survives is the
   grill set — each question then cites WHERE it came from (the file/dataset/gap that
   motivates it).
3. If the topic names a skill (e.g. the sheet-remake wave), read its handoff/census first
   and quote its numbers in the questions.

## Phase 1 — The grill (rounds of 10–20)

- Questions batched under `##` subheaders (theme groups), numbered continuously — never
  one-at-a-time ping-pong unless MG asks.
- Each question: ONE plain sentence of context, then options as plain sentences, then
  **ONE recommendation** (per decision-next-carries-brief). No letter codes, no bare asks.
- Every question names its homework source: `(из: kosha datasets.json — DCS↔CDSL crosswalk)`
  or `(из: census — 10 mega-sheets, consumers unchecked)`.
- Mark the decision type + estimated minutes per question where it is a vote-shaped ask
  (one-word ruling ~1 мин · audit ~3–5 мин · visa ~5–7 мин).
- Themes to actively probe unless clearly out of scope: interlinking (who consumes what),
  data reuse (don't-rebuild inventory), golden standards, pilot readiness, ownership,
  ordering, kill candidates.

## Phase 2 — Transcription (same pass)

MG's answers become: a dated decisions doc in the owning repo (`reports/` or the wave
handoff), GTD rows for every @DO/@DECIDE/@WAITING born from an answer (dates at creation),
and the plan/work updated in the SAME session. Answers are never left chat-only.

## Anti-patterns (each is a defect)

- Asking what a grep would have answered (Phase 0 skipped).
- Questions without options, or options without a recommendation.
- Ten questions where three would do — pad is noise; depth is not count, it is grounding.
- Leaving MG's answers unrecorded at session end.

_Гасунс_
