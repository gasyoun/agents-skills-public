---
name: eli5
description: Explain a topic like I'm a 5 year old. Use when the user types /eli5 <topic> or asks for a dead-simple picture explainer of how something works.
---


# /eli5

_Created: 22-08-2026 · Last updated: 28-08-2026_

Explain like I'm someone who knows nothing about this topic, using an HTML
artifact with big pictures and few words.

Topic: $ARGUMENTS

## Contract

- ONE self-contained HTML file (no external deps, no CDN) shown to the user;
  big inline SVG pictures over prose — at most ~15 words per caption.
- Dead-simple register: no jargon without an immediate picture-level gloss.
- Local/private topics stay local: never publish the artifact; if it describes
  private repo content, it lives in that repo's local-only surface
  (e.g. `dashboard/`), never on Pages.

**This file is the canonical source.** The Codex + Grok twins
(`~/.codex/skills/eli5/SKILL.md`, `~/.grok/skills/eli5/SKILL.md`) are GENERATED
from it — never hand-edit them:
`python ~/.claude/scripts\generate_playbook_twins.py eli5`.

_Dr. Mārcis Gasūns_
