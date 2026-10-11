---
name: "ask-the-wiki"
description: 'Grounded Q&A over the SecondBrain-LLM-Wiki vault — numbered facts first, each with its source note as a wikilink, then a conclusion; the explicit answer «в вики про X ничего нет» is valid when coverage is zero. Uses grep + the link graph (40-Estate/interlinks-dataset.json) as the always-on floor; structured checks probe obsidian-cli, then the obsidian MCP — never fabricate a channel. Triggers: "ask the wiki", "спроси вики", "что в вики про", "find in the vault".'
---

# ask-the-wiki

Use this skill when the user asks to run the migrated source command `ask-the-wiki`.

## Command Template


# /ask-the-wiki <question>

_Created: 02-10-2026 · Last updated: 04-10-2026_

Grounded question answering over the SecondBrain-LLM-Wiki vault (AI Builders peer review item 4, H5632). The contract is the vault's own AGENTS.md rule 1: **нет факта — нет утверждения**. Every fact carries its source note as a wikilink; an explicit zero-coverage answer is a correct answer, an unlinked fact is a fail.

## Paths

Vault root: macOS `<home>/Documents/GitHub/SecondBrain-LLM-Wiki` · Windows `<home>\Documents\GitHub\SecondBrain-LLM-Wiki`. Hand layer: `00-Inbox / 10-Projects / 20-Areas / 30-Resources / 90-Homework`; generated census layer: `40-Estate/`; machine-readable link graph: `40-Estate/interlinks-dataset.json`.

## Procedure

1. **Scope the question** — extract 2–4 search keys; form them in both Russian and ASCII translit/English (notes mix the two).
2. **Grep pass** — `grep -rn -i '<key>' <vault>/ --include='*.md'` over the hand layer first, then `40-Estate/` for generated status notes. Never read or index any `<client>*` path (152-FZ hard deny, vault AGENTS.md rule 9).
3. **Link-graph pass** — for «что связано с X» questions read `40-Estate/interlinks-dataset.json` (`edges[]`: source/target/kind/weight/why; kind «семья»/«карта» = curated, «упоминание» = counted mentions).
4. **Obsidian channels (structured checks; probe before use)** — grep + the link graph are the always-on floor; a live channel is only for exists/backlink/property checks grep is ambiguous about. Probe in this order:
   - **obsidian-cli** first: `obsidian help` — if it answers, use `obsidian search query="…"`, `obsidian read path="…"`, `obsidian property:get …` (parameters `key=value`, boolean flags bare; syntax reference: [skills/obsidian-cli/SKILL.md](https://github.com/gasyoun/claude-config/blob/main/skills/obsidian-cli/SKILL.md)).
   - On the error «Command line interface is not enabled» (Obsidian ▸ Settings ▸ General ▸ Advanced) or an absent binary, fall through to:
   - **obsidian MCP (when live)** — Local REST API, key in `~/.secrets/obsidian-rest-api-key`.
   Name the channel that served, once; if none is live, stay on grep — never fabricate CLI or MCP output.
5. **Compose the answer** in the shape below: numbered facts, one line each, each ending with its source wikilink `[[Note-name]]`; then a short conclusion clearly separated from the facts, never stronger than the facts listed.
6. **Freshness** — check `_Last updated:` on every cited note: older than 30 days = наводка, not факт (lifecycle rule 1); a live probe decides the current state.

## Answer shape

```
Факты:
1. <fact> — [[Note]]
2. <fact> — [[Note]]
Вывод: <only what the facts support>
```

Zero coverage: answer exactly «в вики про X ничего нет» + the keys you tried. Do not pad, do not guess.

_Гасунс_
