---
name: "metadoc"
description: 'Create or update a companion metadoc. Use only when explicitly invoked as /metadoc.'
---

# metadoc

Use this skill when the user asks to run the migrated source command `metadoc`.

## Command Template


# /metadoc

_Created: 08-07-2026 · Last updated: 13-09-2026_

A **metadoc** is a document *about* a document — the durable home for everything you know
*around* an important file: why it exists, who built it, what's still wrong with it, and how
it should evolve. It exists so a "how to improve X" analysis lands next to X and survives the
chat (which evaporates) and any handoff (which gets archived). This command creates or updates
the metadoc for a given subject document.

## Default path (seconds — run this first)

From the Uprava clone root, against the subject path (relative or absolute):

```sh
python tools/metadoc_emit.py <path-to-subject.md>            # dry-run report
python tools/metadoc_emit.py <path-to-subject.md> --apply     # write the stub
```

[`tools/metadoc_emit.py`](https://github.com/gasyoun/Uprava/blob/main/tools/metadoc_emit.py)
(H4097 — the emitter half; [`tools/metadoc_backfill_scan.py`](https://github.com/gasyoun/Uprava/blob/main/tools/metadoc_backfill_scan.py)
is the scanner half, reused here for the exempt/genre classifier — never rebuild it) computes
the real creation date (`git log --follow --diff-filter=A`), reports the subject-token count,
and — with `--apply` — writes the v2 template skeleton to `<stem>.meta.md`. It refuses to
overwrite an existing metadoc (Phase 2b below stays a manual edit). LLM / judgment residue
only after the script report exists — Phase 0 (is this doc long-lived enough to owe a
metadoc?) and filling the skeleton's prose fields stay judgment calls; the script never
guesses those.

This is the invocable half of the standing **"Metadocs — keep a companion record for every
IMPORTANT document"** convention. The **full contract is
[`references/metadocs.md`](https://github.com/gasyoun/claude-config/blob/main/references/metadocs.md)**,
whose H1 is exactly that sentence — H668 moved it out of the resident core for tiered loading,
and the H3871 compression (02-09-2026) left
[`CLAUDE.md`](https://github.com/gasyoun/claude-config/blob/main/CLAUDE.md) carrying only the
one-line rule under § *Changelog, findings, filenames, metadocs* ("Long-lived
manuals/roadmaps/specs keep a sibling `<name>.meta.md`"). Do not go looking for the quoted
heading in the core file; it is not there. That rule fires inline whenever you assess a
document, this command is for doing it deliberately on request.

Two mechanical companions, neither of which this command replaces:

- ⚠️ [`hooks/stop_metadoc_coverage_check.py`](https://github.com/gasyoun/claude-config/blob/main/hooks/stop_metadoc_coverage_check.py)
  — a genre-named doc (`plan`/`roadmap`/`spec`/`memo`/`report`/…) written in a session with no
  sibling `<stem>.meta.md` earns **one reminder per session**. It never blocks, so acting on it
  is on the model.
- [`Uprava/tools/metadocs_index_build.py`](https://github.com/gasyoun/Uprava/blob/main/tools/metadocs_index_build.py)
  — the estate-wide coverage inventory; rebuilds
  [`Uprava/METADOCS_INDEX.md`](https://github.com/gasyoun/Uprava/blob/main/METADOCS_INDEX.md) in
  place, and `--check` exits 1 if the file would change without writing anything.

**Argument:** the subject document — a path (`RussianRamayana/Litpam-Indexator/docs/indesign-pipeline/MANUAL.md`)
or a clear reference to a file discussed in the conversation. If omitted, infer the most
recently analyzed important document from the conversation.

## Phase 0 — qualify the subject (is it metadoc-worthy?)

The threshold is **narrow** — a metadoc is for documents worth maintaining over time:
manuals, roadmaps, specs, references, hub docs, README-of-record. **Not** throwaway notes,
auto-generated files, a trivial one-file repo README that is already self-describing, or
`.ai_state.md` (itself the session journal). Heuristic: *if real effort went into analyzing
how to improve the document, it has earned a metadoc.*

If the subject clearly fails the bar, say so in one line and stop — don't manufacture a
metadoc for a disposable file. If it passes, proceed.

## Phase 1 — locate or scaffold the metadoc

The metadoc is a **sibling** named `<name>.meta.md` next to its subject
(e.g. `MANUAL.meta.md` beside `MANUAL.md`). One metadoc per subject; it travels with it.

- If it already exists → read it, you're **updating** (Phase 2b).
- If not → **create** it with the section skeleton (Phase 2a).

Find the subject's real creation date for provenance:
`git log --follow --diff-filter=A --format=%ad --date=short -- <subject> | tail -1`
(convert ISO → `DD-MM-YYYY`). If untracked, use today.

## Phase 2a — author a new metadoc

Fill this skeleton (Russian or English to match the subject's language; the example subject
`MANUAL.md` is Russian, so its metadoc is Russian):

```markdown
# <name>.meta.md — метадокумент о `<name>`

_Created: DD-MM-YYYY · Last updated: DD-MM-YYYY_

Это **метадокумент** — документ *о документе*. Его предмет — [<name>](<full blob URL>).
Здесь не дублируется содержимое субъекта, а фиксируется всё, что *про* него. Ведётся по
общему правилу «для каждого важного документа — свой метадокумент» (см. `~/.claude/CLAUDE.md`).

## Предмет
- **Документ:** [<name>](<full blob URL>)
- **Назначение:** <one-line what-it's-for>
- **Аудитория:** <who reads it>
- **Формат / контракт:** <MD contract, any code/name contract it must obey>

## Провенанс
- **Создан:** <DD-MM-YYYY> (<handoff H### / session>, <model tier + exact version>).
- **Следующая закалка:** <planned handoff, or "нет">.

## Бэклог улучшений (ранжированный)
<table: # · Улучшение · Почему · Статус (queued (H###) / ✅ + PR link)>

## Известные ограничения / оговорки
<scope limits, guardrails, caveats>

## Intended use / known misuse
<what the subject is FOR; the known wrong ways to use/read it — datasheet/model-card field>

## Maintenance & sunset plan
<who/what keeps the subject alive (owning repo, pipeline, human); what "archived/ended" looks like — Endings Project field>

## Deprecation status
<`active` | `superseded by [X]` | `retired` — default `active` — Programming Historian field>

## Связанные документы
<full blob/tree links to sibling docs — the "which one do I open" map>

## История ревизий метапредмета
<table: Дата · Событие · Кто (tier+version)>

_Dr. Mārcis Gasūns_
```

The **improvement backlog is the heart of the metadoc** — this is where any "how to improve X"
analysis from the conversation goes. Rank items; give each a one-line "why"; mark status
(`queued (H###)` when a handoff owns it, `✅ + PR link` when closed). If the analysis is large
enough to be its own build, mint a handoff for it (per the always-mint rule) and cite it in the
Status column rather than leaving the backlog ownerless.

## Phase 2b — update an existing metadoc

- Add newly-identified improvements to the backlog; **don't** restate ones already there.
- Tick closed items to `✅` with their PR link; move nothing out — the backlog is a ledger.
- Append a **revision-history row** for whatever changed in the subject since last update.
- Bump `Last updated:` to today; leave `Created:` alone.
- If the subject was substantially revised this session, that revision IS a history row —
  record it.

## Phase 3 — persist & commit

- Authored-MD contract: dated header, `_Dr. Mārcis Gasūns_` byline, **full `blob`/`tree`
  URLs** (never relative — the metadoc gets pasted into chats/issues), plain Markdown (no
  HTML), `DD-MM-YYYY` dates, model attribution as **tier + exact version**.
- Commit per the repo's git discipline: if a handoff already scopes the work (or the
  improvement backlog just minted one), commit → PR → merge with no confirmation ask; for the
  private `Uprava` hub, direct push to `main`. Public repos go through a PR with auto-merge.

## Guardrails

- **A metadoc is not a copy of the subject.** If you find yourself restating the subject's
  content, stop — the metadoc holds only what's *around* it (why/gaps/history), never a
  duplicate of *what* it says.
- **One metadoc per subject**, sibling-named. Don't scatter improvement notes across
  `.ai_state.md`, handoffs, and chat when a metadoc is the canonical home.
- **Narrow threshold** (Phase 0) — resist making metadocs for disposable files; a metadoc for
  everything is as useless as a metadoc for nothing.
- **Backlog is a ledger, not a scratchpad** — closed items stay (marked ✅ + PR), they don't
  get deleted, so the metadoc doubles as the document's improvement history.
- **Every open backlog row must cite an owner** — an `H###` handoff or an explicit `parked`
  verdict. An ownerless "open" row is a defect, not a valid state — either mint the handoff
  it deserves or mark it `parked` with a one-line reason.
- Never put secrets, tokens, or unpublished personal data into a metadoc (it may be public).

_Dr. Mārcis Gasūns_
