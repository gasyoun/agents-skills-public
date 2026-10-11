---
name: "readme-truth-pass"
description: 'Rewrite a README (or small repo set) to verified reality — live counts, actual tool/file inventory — OR drain a pre-catalogued roadmap/checkbox/status drift list. Triggers: "readme truth pass", "fix stale roadmap checkboxes".'

---

# readme-truth-pass

Use this skill when the user asks to run the migrated source command `readme-truth-pass`.

## Command Template


# /readme-truth-pass <repo|repo-list|group> [--mode readme|roadmap-drift]

_Created: 17-07-2026 · Last updated: 26-08-2026_

The mechanical README-refresh discipline distilled from [H548](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H548-Opus_Uprava_readme-refresh-dict-repos_10.07.26.md) (Opus 4.8 — org-wide dict-repos audit+refactor, one PR per repo) and [H283](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H283-Fable_SanskritLexicography_russiantranslation_readme_timeline_usecases_07.07.26.md) (Fable 5 — RussianTranslation researcher-first rewrite + milestone timeline). Sits between two existing skills: narrower than [`/workspace-manual`](https://github.com/gasyoun/claude-config/blob/main/commands/workspace-manual.md) (which authors a whole audience-split manual set) and heavier than [`/md-hygiene`](https://github.com/gasyoun/claude-config/blob/main/commands/md-hygiene.md) (which only fixes header/byline/HTML/link FORM, never rewrites content to match reality) — this rewrites the README's CONTENT to verified truth, applying `/md-hygiene`'s form rules along the way.

**Skill-mine C29 (H1779–H1935, MG vote 30-07-2026: UPDATE this skill, not NEW `/roadmap-drift-sweep`):** `--mode roadmap-drift` owns the multi-repo **roadmap / checkbox / status-banner drift** drain (H1877–H1880) that `/hub-staleness-audit` does not cover (hubs only) and that bare README refresh does not cover (ROADMAP / ROADMAP_INDEX / status banners).

**Mechanical — Sonnet 5 (`claude-sonnet-5`) tier.** A repo whose pass needs a genuine audience reframe (the H283 researcher-first shape, not just fact-refresh) may warrant Fable 5 (`claude-fable-5`) or Opus 4.8 (`claude-opus-4-8`) for that repo specifically — the census evidence used both tiers depending on how much rewrite-judgment the repo needed.

## Argument contract

**Default mode `readme`:** one repo, a space/comma-separated list of repos, or a named group (`dict-repos` / `tooling-repos` / `research-repos` — the H548/H549/H550 split, Opus 4.8 ×3: [H549](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H549-Opus_Uprava_readme-refresh-tooling-repos_10.07.26.md), [H550](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H550-Opus_Uprava_readme-refresh-research-repos_10.07.26.md)). For a group, re-derive the live repo list at execution time (`gh repo list <org> --limit 200 --json name,isArchived,pushedAt -q '.[] | select(.isArchived==false) | .name'`) — never trust a frozen snapshot, repos get added/archived between sessions. No argument = the repo you're cd'd into.

**`--mode roadmap-drift`:** a pre-catalogued drift list (path to harvest/staging doc or handoff that names repos + items). Example: [ASK_BATCH_STAGING_ROADMAP_HARVEST_2026-07.md](https://github.com/gasyoun/Uprava/blob/main/archive/ASK_BATCH_STAGING_ROADMAP_HARVEST_2026-07.md). Optional cluster filter (Cologne / grammar-dh / tier0-ru / narrative-hub). **No catalogue named → generate one** (RD Phase 0a).

## Mode: `roadmap-drift` (C29)

Drain a **pre-catalogued** multi-repo drift list — not invent a new axis (`/org-axis-sweep`) and not re-ground hub prose only (`/hub-staleness-audit`).

### RD Phase 0a — Generate the catalogue when none was named (H3075, Opus 5 — hub-residual W1 gate + roadmap handoff truth check)

Run the drained-roadmap check over the whole registered estate:

```
python ~/Documents/GitHub/Uprava/tools/roadmap_handoff_truth.py --check --md
```

Exit **0** = no drained roadmap above threshold; say so and stop. Exit **1** = its Markdown table **is** the catalogue — each row is a roadmap in [ROADMAP_INDEX.md](https://github.com/gasyoun/Uprava/blob/main/ROADMAP_INDEX.md) whose every H### reference has closed, i.e. a page that reads as a plan while all its units have shipped. Feed those rows straight into RD Phase 1.

**Report, don't rule.** A drained roadmap is not automatically wrong — rulings survive their backlog. Per item the verdict is *truth-pass in place* or *archive with a tombstone*, decided by the three-part rule in [ARCHITECTURE_UPRAVA_HUB_RESIDUAL_2026H2.md](https://github.com/gasyoun/Uprava/blob/main/docs/ARCHITECTURE_UPRAVA_HUB_RESIDUAL_2026H2.md). Either way, leave the dated `> **Truth-pass DD-MM-YYYY**` banner on the page — that banner is what suppresses the flag next run, so a pass that fixes the prose but omits the banner will re-flag forever.

Threshold, baseline, and the check's recall limit (a single never-flipped registry row masks an otherwise fully drained page): [ROADMAP_TRUTH_CHECK_DRAINED_ROADMAP_BASELINE_18-08-2026.md](https://github.com/gasyoun/Uprava/blob/main/docs/ROADMAP_TRUTH_CHECK_DRAINED_ROADMAP_BASELINE_18-08-2026.md). Default `--min-done 6`; `--json` for a snapshot to diff against the baseline. This check answers only "have all this page's handoffs closed" — open checkboxes stay [`roadmap_drain_report.py`](https://github.com/gasyoun/Uprava/blob/main/tools/roadmap_drain_report.py)'s question.

### RD Phase 0 — Load catalogue

1. Open the named harvest/staging/handoff (or the RD Phase 0a table). Extract every item: repo · file/path · claim · suggested fix (if any).
2. Group by repo. One PR per repo (or direct push where the repo takes them). Never touch `csl-orig` dictionary text.
3. Re-verify against **current `origin/<default>`** in a fresh worktree or after `git fetch` — catalogue rows age fast; at least one H1877-family item was wrong on re-check.

### RD Phase 1 — Per item

For each catalogued item:

1. Locate the row/checkbox/banner in the live file on `origin`.
2. Gather evidence (merged PR, commit, file exists, handoff ✅).
3. Verdict:
   - **Fix** — edit the roadmap/status surface so it matches reality (flip checkbox, rewrite banner, strike dead plan with pointer).
   - **Still accurate** — leave content; write a one-line evidence note in the handoff completion table (do not invent edits).
   - **Catalogue wrong** — correct the harvest row / ROADMAP_INDEX pointer; do not invent the missing plan.

### RD Phase 2 — Deliver

- Completion table: item → verdict → evidence link.
- One PR per repo; `/pr-babysit` for the batch.
- Not a substitute for full README rewrite — if both are needed, run `readme` mode after or as a separate session.

Evidence: [H1877](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1877-Sonnet_multi_roadmap-drift-sweep-cologne-cluster_29.07.26.md) (Sonnet 5 — Cologne) · [H1878](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1878-Sonnet_multi_roadmap-drift-sweep-grammar-dh-cluster_29.07.26.md) (Sonnet 5 — grammar/DH) · [H1879](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1879-Sonnet_multi_roadmap-drift-sweep-tier0-ru-cluster_29.07.26.md) (Sonnet 5 — Tier-0 RU) · [H1880](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1880-Sonnet_multi_roadmap-drift-sweep-narrative-hub-cluster_29.07.26.md) (Sonnet 5 — narrative/hub).

---

## Mode: `readme` (default)

## Phase 1 — Gather live facts (never carry forward the old text)

Per repo, before touching the README:

- **Status** — is the thing it describes actually working/current: open rows in `Uprava/GTD_NEXT_ACTIONS.md`, the handoff registry (search the repo name in [`Uprava/handoffs/README.md`](https://github.com/gasyoun/Uprava/blob/main/handoffs/README.md)), the repo's own `.ai_state.md`/`CLAUDE.md` if present.
- **Counts** — every file/entry/dataset-size claim in the current README, re-verified (`wc -l`, actual directory listings) — a carried-forward stale number is worse than no number.
- **Tool/inventory** — `tools/`/`src/` script census, the actually-current CLI surface.
- **Recent history** — `git log --oneline -20`, recently merged PRs, to catch a milestone the README hasn't caught up to yet.

## Phase 2 — Rewrite

Apply the org doc contract (delegate the mechanical half to `/md-hygiene` if the repo needs a broader multi-doc sweep too):

- Dated header with the REAL creation date — `git log --follow --diff-filter=A --format=%ad --date=short -- README.md | tail -1`, converted to `DD-MM-YYYY`; never stamp today's date as "Created" on an existing file.
- Byline `_Dr. Mārcis Gasūns_`, no raw HTML (a blockquote `> ` instead), every path/URL rendered as a full `blob`/`tree` Markdown link.
- Audience framing matched to the repo kind:
  - **Dict-repos** (one Cologne dictionary's text/data): export status, source scan, issue-taxonomy table, a link to `csl-orig`'s canonical correction-workflow doc instead of re-describing it inline.
  - **Research/data repos**: researcher-first — project, method, datasets (FAIR table if applicable), consumer use cases, THEN operator quickstart (the H283 shape).
  - **Tooling repos**: install + API/CLI surface first.
- Spot-check a sample of the README's links actually resolve (404s creep in after file moves/reorgs).

## Phase 3 — csl-orig guardrail

A README refresh in a dictionary repo is normal repo maintenance, NOT the text-correction batch-PR pipeline — ships as a regular PR (or a regular commit in a repo that takes direct pushes), never routed through `/cologne-batch-pr`. Don't confuse README hygiene with the "never commit dictionary text corrections directly to `csl-orig`" rule — that rule is about the dictionary source text, not its README.

## Phase 4 — Deliver

One PR per repo — READMEs in different repos can't be combined into one PR. Multi-repo runs: fan out via a `pipeline()`/`parallel()` Workflow or an `Agent` fleet, one agent per repo doing gather → rewrite → PR; cap concurrency sensibly. After the fan-out completes, run [`/pr-babysit`](https://github.com/gasyoun/claude-config/blob/main/commands/pr-babysit.md) to merge the green batch in bulk rather than merging each by hand. A repo whose "current state" is genuinely ambiguous (two contradictory handoffs both claim to be the latest status) becomes a GTD `@DECIDE` row, not a guess — continue with the rest of the batch.

## Definition of done

- Every target repo has either an updated README (merged PR) or an explicit skip reason (archived, already current — note which).
- `/pr-babysit` run once to sweep the resulting PR batch.
- `Uprava/GTD_NEXT_ACTIONS.md` updated with the outcome, or a follow-up `@DECIDE` row for any repo that needed a human call.
- Handoff closed via [`/handoff-close --reconcile`](https://github.com/gasyoun/claude-config/blob/main/commands/handoff-close.md) (H1342 (Opus 4.8) — bookkeeping batching and WIP cap — the registry row flip + close clause buffer into the session's single `registry_reconcile.py flush`), model tier + exact version stated.

## Gotchas

- Re-verify every count — a README that carries forward an old number reads as more authoritative than saying nothing, and is the specific defect H548 was minted to fix.
- `/md-hygiene` alone is not enough here — it fixes form (header/byline/HTML/links) but never checks whether the CONTENT is still true; this skill's whole job is the content check `/md-hygiene` skips.
- Don't route a README-only PR through the dictionary correction-queue pipeline (Phase 3).

Evidence: [H548](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H548-Opus_Uprava_readme-refresh-dict-repos_10.07.26.md) (Opus 4.8), [H283](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H283-Fable_SanskritLexicography_russiantranslation_readme_timeline_usecases_07.07.26.md) (Fable 5). Roadmap-drift mode: skill-mine C29 vote 30-07-2026 ([H1937](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1937-Grok_Uprava_skill-mine-h1779-h1935_30.07.26.md), Grok 4.5 — skill-mine H1779–H1935).

_Dr. Mārcis Gasūns_
