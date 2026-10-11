---
name: "document"
description: 'Sweep registration surfaces for shipped deliverables. Use only when explicitly invoked as /document.'
---

# document

Use this skill when the user asks to run the migrated source command `document`.

## Command Template


# /document

_Created: 11-07-2026 · Last updated: 01-10-2026_

(ex `/artifact-propagate` — renamed 25-08-2026; the old name stays as an alias stub command at [`commands/artifact-propagate.md`](https://github.com/gasyoun/claude-config/blob/main/commands/artifact-propagate.md), and BOTH names count as sweep evidence everywhere. Dual-run note: two sessions landed overlapping renames the same morning — 25-08-2026 adjudication merged both halves into this runbook.)


## Default path (seconds — run this first)

```sh
python ~/Documents/GitHub/Uprava/tools/document_check.py <artifact>[::kind] [<artifact2>[::kind] ...] --repo <repo> [--kind doc|dataset|interface|skill|paper] [--receipt [PATH]] [--force] [--handoff H####]
```

(`--kind` defaults to `doc`; the Phase 0 table names the kinds. Per-artifact `path::kind` overrides `--kind` for THAT artifact, so a mixed batch is gated correctly; `--reason` reasons are GLOBAL to the whole batch (not per-artifact — only the kind is per-artifact); an unknown `--reason` id is warned loud with the valid-id list (H4545) — a warning means your reason did NOT apply.) LLM / judgment residue only after the script report exists (or the script exits non-zero with a real defect). Rights / MEGABOOK / n-a reasons stay after that report.

**Batch mode:** several artifact paths in one invocation = one table each plus combined totals — use it when a session shipped more than one durable deliverable (the post-`/ask-batch` shape) so the sweep is one pass.

Born 10-07-2026 from the CommentaryStrategies role-guides session ([H497](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H497-Fable_CommentaryStrategies_leonov_sundarakanda_guide_10.07.26.md) / [H517](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H517-Fable_CommentaryStrategies_kostina_sundarakanda_guide_10.07.26.md) / [H533](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H533-Fable_CommentaryStrategies_gasuns_sundarakanda_guide_10.07.26.md), all Fable 5 — Leonov / Kostina / Gasūns Sundarakāṇḍa role guides): after the
deliverable shipped, it took **nine consecutive user prompts** ("а в MANUAL?", "а в README?", "а в
GTD?", "а в kosha?"…), and *every single one found a real gap*. The knowledge existed once — as a
checklist it costs one pass; as user-driven archaeology it costs a prompt per surface. This skill
IS that checklist, run proactively. (Changelog/release added to the list at MG's request, 11-07-2026.)

**Mechanical reminder (24-07-2026, after [H1483](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1483-Opus_SanskritLexicography_definition-typology-classifier-ws2-4_22.07.26.md) (Opus 4.8) — definition-typology classifier WS2-4; widened 28-07-2026 by [H1805](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1805-Fable_claude-config_propagation-trigger-hardening_28.07.26.md) (Fable 5) — propagation-trigger hardening):** Stop hook
[`stop_document_check.py`](https://github.com/gasyoun/claude-config/blob/main/hooks/stop_document_check.py)
blocks once per session on TWO triggers: (1) durable artifact paths dirty (`data/`, dated
reports, TSV/CSV) with no registration surface touched; (2) **a handoff was CLOSED this
session** (transcript shows `handoff_close.py` / "DONE: H### closed") **but no propagation
sweep is in evidence** — neither this skill's name (current or legacy), nor a
`document_check.py` invocation, nor a dated `document_sweep_*` receipt, nor ≥4 distinct
surface names appear in the transcript. Trigger 2 exists because the worktree-delivery shape reaches Stop with a
clean tree, and on 28-07-2026 that cost the user TEN consecutive «а в X отражено?» prompts
after the [H1761](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1761-Fable_CommentaryStrategies_sundara-commentary-guidelines-reader-contract_27.07.26.md) (Fable 5 — Sundara commentary-guidelines reader contract) / [H1788](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1788-Fable_Uprava_mint-registry-row-tree-guard-ci-sweep_28.07.26.md) (Fable 5 — mint registry-row tree-guard CI sweep) closes — every one found a real gap. **Running this skill is the
recovery.** Pair with [`/cut-release`](https://github.com/gasyoun/claude-config/blob/main/commands/cut-release.md) when the changelog was edited.

Sibling boundaries: [`/findings-append`](https://github.com/gasyoun/claude-config/blob/main/commands/findings-append.md) routes one *finding* to its one-two
homes; [`/hub-staleness-audit`](https://github.com/gasyoun/claude-config/blob/main/commands/hub-staleness-audit.md) sweeps hubs for *stale old* claims. This
skill is artifact-centric and *breadth-first*: one deliverable × all surfaces, now.

**Argument:** the artifact(s) (path or name) + optionally their repo. Infer the kind from the file
itself; if ambiguous, ask ONE question, not four.

**Start with the script ([H2797](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H2797-Sonnet_Uprava_zj-artifact-propagate-checklist_15.08.26.md), Sonnet 5 — artifact-propagate checklist script, 15-08-2026):** most cells below are file-exists /
row-mentions-artifact checks — mechanical, not judgment. Run
[`Uprava/tools/document_check.py`](https://github.com/gasyoun/Uprava/blob/main/tools/document_check.py)
first (the *Default path* invocation above, with `--kind <kind>` from Phase 0) and use its
present/missing/n-a table as the starting checklist instead of opening every hub from memory.
It encodes the full surface list below as config — 8 repo-local + the 18 org hubs (H1805's 17
+ `reuse_index`, H4545);
the Phase 1–2 tables ARE that config, kept in lockstep by `--check-doc` — so a
dropped row is a script defect, not a recall gap. A relative artifact path given together with `--repo` resolves against the repo root first (the [H3251](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H3251-Grok_Uprava_artifact-propagate-repo-cwd_21.08.26.md) fix, Grok 4.6 — artifact-propagate `--repo` cwd resolution), so running from any cwd is safe. What it does NOT decide, and what stays this skill's job:

- whether a `missing` surface genuinely applies here (rights / tier / "does this belong"
  content judgment) — the script reports the cell, a human/agent supplies the verdict;
- `n/a because ...` reasons for judgment-mode surfaces (`sibling_domain_docs`, `issues`,
  `megabook`, `epistemic_registries`, `danger_facts`) — pass `--reason id="..."` once you
  have decided, never let the script invent one;
- MEGABOOK thesis edits — always hand-authored, never scripted.

`--selftest` proves the fixture; `--json` for a machine-readable pass; `--check-doc`
verifies this runbook still names every surface id/label and the stated 8/18 counts (run it
after editing either side — drift exits 1); `--receipt [PATH]` writes a dated
`document_sweep_<YYYY-MM-DD>_<artifact-stem>.json` verdict file (default location: the `--repo`
root; the ISO date is what the Stop hook's receipt regex matches; a same-day second sweep gets
its own name — an existing receipt is never overwritten without `--force`, [FINDINGS §560](https://github.com/gasyoun/Uprava/blob/main/FINDINGS.md), H4545; the payload
carries provenance — repo git HEAD/branch, host, `--handoff H####`) — **commit it with the sweep edits**: it is the
durable evidence a later weekly review or Stop-hook trigger can cite instead of transcript
archaeology. Missing surfaces still get closed (or given a `--reason`) via Phases 1–2 below — the script only replaces
the *discovery* legwork, not the closing.

---

## Materiality gate — pick the sweep speed BEFORE Phase 0
## Phase 0 — classify the artifact

| Kind | Examples | Surfaces that CAN apply |
|---|---|---|
| **doc / manual / guide** | role guide, MANUAL, runbook, spec | all repo-local + GTD + INTERLINKS (if it changes who-does-what) |
| **derived dataset** | apparatus JSON, crosswalk, frequency table | repo-local + kosha manifest + DATA_LAYERS_CENSUS + FEATURES_INDEX + INTERLINKS |
| **interface / site / page** | dashboard, review sheet, HTML apparatus | repo-local + FEATURES_INDEX + INTERLINKS |
| **skill / command** | a new `~/.claude/commands/*.md` | SKILLS_INDEX + Codex twin ([`/claude-codex-skill-porter`](https://github.com/gasyoun/claude-config/blob/main/commands/claude-codex-skill-porter.md)) — NOT FEATURES_INDEX |
| **paper / draft** | Axx manuscript | [`/articles-update`](https://github.com/gasyoun/claude-config/blob/main/commands/articles-update.md) owns it; only .ai_state + GTD here |

## Phase 1 — repo-local surfaces (8 repo-local rows, walk ALL, in this order)

When the artifact spans several repos, walk per repo in this order: home repo first, then
Uprava org hubs (Phase 2), then kosha/spine registries — worktree discipline per Phase 4
applies to EACH repo touched, not just the home one.

Ids match the script config (`--reason`/verdict keys):

| id | Surface | What to do |
|---|---|---|
| `readme` | README.md | Does the section owning this topic mention the artifact? Add a row/line, not an essay. |
| `claude_md` | CLAUDE.md | Do agents need to know it exists, and is there a *sync rule* to state (e.g. "change X ⇒ update Y in the same PR")? The behavioral rule is worth more than the bare link. |
| `cold_start` | Cold-start doc (GEMINI.md / equivalent) | Orientation table row with a "Read when" column. |
| `sibling_domain_docs` | Sibling domain docs (manual/roadmap/method docs) | Put the pointer where the *reader already is* (e.g. next to a "[на ратификацию]" status note), not in a generic "links" dump. Historical/decision-record docs get a short pointer block + a **precedence note** ("where they disagree, X wins"); out-of-scope records get skipped — say so. |
| `metadoc` | Metadoc (\<name\>.meta.md) | Important doc without one → create it; subject already has one → add the revision-history row. If the governing doc has no metadoc yet, this is the moment it gets one. |
| `changelog` | CHANGELOG.md -> release (/cut-release) | Entry under `[Unreleased]`, then the mandatory [`/cut-release`](https://github.com/gasyoun/claude-config/blob/main/commands/cut-release.md) follow-through: promote, tag, release, CITATION.cff version+date sync (stale CITATION.cff is a recurring find — check even untouched). Watch the case-sensitivity trap: tracked file may be `changelog.md` lowercase — `git add CHANGELOG.md` on Windows silently stages nothing. |
| `ai_state` | .ai_state.md | ✅ Completed entry AND a Next-Steps/WIP/@WAITING refresh if the artifact changes what happens next (stale queue rows are the norm, not the exception). |
| `issues` | Issues (per-person task issues) | Comment with the link on affected ones (external-facing: needs standing or per-action authorization; if blocked, park ready-to-paste text in GTD as @DO). |

## Phase 1b — manuals freshness (added 08-09-2026, H4416)
## Phase 2 — org hubs (walk ALL; most verdicts will be "not applicable")

All 18 org surfaces below carry the same `| \`id\` |` row contract the script's
`--check-doc` parity gate enforces (H4545b row-anchored: a dropped row is a red gate,
not a style nit). When in doubt, verdict `n/a` with a one-line reason beats invention.

| id | Surface | What to do |
|---|---|---|
| `gtd` | Uprava/GTD_NEXT_ACTIONS.md | Strike the @DO row / append the new one; human tasks are table rows, never prose. |
| `handoffs_registry` | Uprava/handoffs/README.md | The mint/close pipeline usually already wrote it — verify, don't re-write. |
| `features_index` | SanskritLexicography/FEATURES_INDEX.md | One row per feature when the artifact is one; n/a for docs-only work. |
| `reuse_index` | Uprava/REUSE_INDEX.md (reusable assets) | Reusable structure/asset → row with the consumer contract; questions go to /prior-art. |
| `kosha_manifest` | kosha/data/manifest/datasets.json | Dataset intake/change → manifest row, machine-checkable. |
| `data_layers_census` | Uprava/DATA_LAYERS_CENSUS.md | New/renamed data layer → census row; renames route through /descriptive-rename. |
| `project_interlinks` | Uprava/PROJECT_INTERLINKS.md | Ownership/dependency edges between repos changed → update the map. |
| `skills_index` | root SKILLS_INDEX.md | New/changed skill → registration row; a doc sweep is usually `n/a` (catalog, not doc hub). |
| `articles` | Uprava/ARTICLES.md | Paper/manuscript milestone → row; manuscripts live private-only per the 04-10 ruling. |
| `danger_facts` | Uprava/DANGER_FACTS.md + context_facts.yaml | Destructive-risk fact → BOTH the store and the repo's own memory per /danger-memory. |
| `questions_log` | Uprava/QUESTIONS_LOG.md | Open question the artifact created/answered → log it with the decision link. |
| `megabook` | Uprava/MEGABOOK.md | Thesis-level synthesis only (one line); routine units are `n/a`. |
| `skills_manual_ru` | Uprava/SKILLS_MANUAL_RU.md | RU-facing manual chapter affected → update the chapter, not the index. |
| `hooks_index` | claude-config HOOKS_INDEX.md | Hook added/changed/unwired → row + 🛑/🔁/⚠️ class. |
| `repo_status_rollup` | Uprava/REPO_STATUS_ROLLUP.md | Repo-level status shift (archived/frozen/rebooted) → rollup row. |
| `hub_manifest` | Uprava/tools/hub_manifest.json | Hub file added/removed → manifest entry, else the hub-audit drifts. |
| `auto_derived_views` | Auto-derived views (NEXT_UP/BOTTLENECKS/kanban/dashboards/HUB_STALENESS_REPORT) | NEVER hand-edit — regenerate via the owning tool and verify sources instead. |
| `epistemic_registries` | FINDINGS x2 + 7 epistemic registries (ASSUMPTIONS/CONTRADICTIONS/GAPS/DEAD_ENDS/RECIPES/STALENESS/GLOSSARY, Uprava + SanskritLexicography) | The one registry that changed → append via /findings-append; never bulk-rewrite. |

## Phase 3 — negative verdicts are deliverables too

For every surface skipped, write ONE line of *why* in the final report ("SKILLS_INDEX — каталог
скиллов, не документов; не место"). The session that spawned this skill proved the "no" answers
(SKILLS_INDEX, FEATURES_INDEX-as-docs) take judgment too — recording them stops the next session
from re-litigating, and stops well-meaning inventory noise.

## Phase 4 — delivery mechanics

- **Contended shared trees** (Uprava, SanskritLexicography, kosha during multi-session bursts):
  deliver via a fresh `git worktree` from `origin/<default>` and push from there; never fight
  another live session's checkout, staged files, or branch. Remove the worktree + branch after.
- PR-per-repo (merge in the same pass, per the handoff-autonomy rule); private `Uprava` takes
  direct pushes to `main`.
- Authored-MD contract everywhere: full blob URLs, `DD-MM-YYYY` header bumps, no HTML.
- Commit the `--receipt` JSON alongside the sweep edits.
- **Final report:** one table `surface → ✅ added / ✏️ updated / ⛔ n/a (reason)` — every surface
  from Phases 1–2 present, none silently skipped. If >3 surfaces changed, the table also goes into
  the handoff file.

## Phase 5 — sweep findings become backlog items (added 25-08-2026)

Defects found WHILE sweeping do not evaporate into chat — each one lands before the sweep ends:

1. Missing/stale metadoc backlog item → the metadoc's ranked backlog (the how-ai-marketer precedent, 24-08-2026).
2. A stale hub row this sweep can't safely fix inline (money contour, foreign lane, needs MG ruling) → GTD `@DO`/`@DECIDE` row, owner + date.
3. A repeated-question pattern surfacing again → `questions_log` Reps bump (that IS the pattern's counter).
4. `--check-doc` drift (config ↔ this runbook diverged) → fix BOTH sides in the same pass; the gate exists so drift is loud, not so drift is tolerated.

## Exit gate — prove closure (added 25-08-2026)

Re-run the script at the END of the sweep:

```sh
python ~/Documents/GitHub/Uprava/tools/document_check.py <artifact> --repo <repo>
```

Pass = ZERO `missing` cells remain unexplained — every row is either ✅ (closed in
Phases 1–2) or carries an explicit n/a reason. Capture the before/after cell counts from
`--json` (run it both ends) into the final report table; a sweep whose numbers cannot be
shown closed is an open sweep, whatever the prose claims. Idempotent by design: re-running
costs seconds and catches the surface you edited but did not save, or the hub another
concurrent session moved mid-pass. Add `--receipt` to this final re-run: the dated
`document_sweep_<YYYY-MM-DD>.json` it writes IS the closure proof — commit it per Phase 4
(or as the delivery pass' follow-up commit if the receipt is written after Phase 4 already
pushed).

_Dr. Mārcis Gasūns_
## References

Detail moved verbatim (read on demand — lives in the rendered skill copy's references/):

- `references/materiality-gate.md` — Materiality gate — sweep-speed decision detail
- `references/phase1b-manuals.md` — Phase 1b manuals freshness (H4416)
- `references/phase2-org-hubs.md` — Phase 2 org-hub walk detail — hub-by-hub verdicts
