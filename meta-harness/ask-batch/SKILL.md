---
name: "ask-batch"
description: '/ask batch mode - stage layered plans and tier-locked H### handoffs across every repo worked in a recent window, in a resumable manifest. Sittings run as published review-sheets; chat only when the human sits. Triggers: "ask-batch", "apply /ask to all my repos". Opus 4.8 (claude-opus-4-8)/Fable 5 (claude-fable-5) interviews.'
---

# ask-batch

Use this skill when the user asks to run the migrated source command `ask-batch`.

## Command Template


# /ask-batch [--since <30d>] [--repos a,b,c] [--tier 0|1|all] [--slice N] [--resume]

_Created: 13-07-2026 · Last updated: 05-09-2026_

The breadth counterpart to `/ask`. `/ask` goes deep on ONE repo in one sitting; `/ask-batch`
goes wide — it takes every repo you touched in a window and stages, for each, the same full
layered plan plus a batch of **queued, tier-locked** handoffs, so that when any of that work
actually activates the plan and the tasks are already sitting there ready to launch. The
deliverable is a pre-staged backlog, not execution: **nothing minted here runs** until you
(or `/next-task`) pick it up later.

Because a full plan + a handoff pile across ~30 repos cannot finish in one session — and
because you answer the interviews in sittings, not all at once — the skill is built around a
**resumable manifest**. Every phase records its state there; any session resumes with
`--resume` and continues from the first unfinished repo/phase.

**Arguments** — `--since` (window, default `30d`); `--repos` (explicit comma list, skips
discovery); `--tier` (limit to priority Tier-0/1 repos, default `all`); `--slice N` (process
only the next N repos this pass — the proving/pacing lever); `--resume` (continue an existing
manifest instead of starting one).

## The resumable manifest

One control doc: [`Uprava/archive/ASK_BATCH_STAGING_2026-07.md`](https://github.com/gasyoun/Uprava/blob/main/archive/ASK_BATCH_STAGING_2026-07.md)
(the July doc lives in `archive/` — archived per its own retention rule below; a fresh
span's staging doc starts at the Uprava root). A table, one row per in-scope repo, with
columns: priority tier · audit ✅/– · fork-shortlist count · interview ✅/– · rulings recorded ·
plan docs ✅/– · handoffs minted (count + tier split) · gate verdict · notes. Dated header +
byline. Every phase below updates this row for its repo in the same pass it does the work, so a
crashed or paused session loses nothing and `--resume` is exact. This doc is the source of truth
for "what's staged and what's left".

**Retention (H3101 (Sonnet 5), stale-docs snapshot/archive sweep, 19-08-2026):** once every handoff a staging doc references is closed (none
of its `H###` rows resolve to a live file under `Uprava/handoffs/`), `git mv` it to
[`archive/`](https://github.com/gasyoun/Uprava/tree/main/archive) — a finished staging doc is a
snapshot of what was true when it ran, not a registry. Don't prune a doc still feeding an open
handoff.

## Phase 0 — Discover the repo set

Determine "repos worked in last <window>" by the **union** of:

- **Git authorship** — commits by the user's git identity (`user.email`) in the window, across every clone under `~/Documents/GitHub/`.
- **Activity signals** — repos named in [`Uprava/GTD_NEXT_ACTIONS.md`](https://github.com/gasyoun/Uprava/blob/main/GTD_NEXT_ACTIONS.md) rows touched in the window, repos with a window-dated handoff in [`Uprava/handoffs/`](https://github.com/gasyoun/Uprava/blob/main/handoffs/README.md), and repos whose `.ai_state.md` changed in the window.

Rank by the standing priority tiers (Tier-0 student/revenue → Tier-1 research → Tier-2), annotate each with its last-active date and why it's in scope, and **present the list for confirmation/prune** before anything is planned — discovery is cheap, a wrong repo set wastes a lot of downstream authoring. Write the confirmed set as the manifest's initial rows.

## Phase 1 — Audit every in-scope repo (read-only, fan-out-friendly)

For each repo, run `/ask` Phase 1 (which includes `/prior-art`): read `.ai_state.md`/README/existing
roadmaps/CHANGELOG, open issues/PRs (verify owner via `git remote -v`), git log since the last
plan, and the repo's hub rows (`PROJECT_INTERLINKS`, `REUSE_INDEX`, `SHARED_CODE`, `ARTICLES`).
Produce each repo's private fork shortlist — the genuine decisions its plan will hinge on. This
phase is pure reads and parallelizes well (an `Explore` agent per repo, or the workflow fan-out
in the note below). Record each repo's fork-shortlist count in the manifest.

### Every fan-out agent gets an early-return budget — non-negotiable

A read-heavy audit agent left unbounded does not fail loudly, it **stalls**: the harness waits
180 s past its last progress and then restarts it **from zero**, so a dying agent burns its full
run and returns nothing. Measured 27-07-2026 on the Fable roadmap harvest: round 1 lost **4 of 7
clusters**, one Systema agent burning 854 k tokens across six attempts for no result. Adding the
budget below to the prompt cut the failure rate to **1 of 6** and finished the same coverage in a
third of the tokens.

So every harvest/audit/verify agent prompt carries, in the prompt body and not merely in the schema:

```
## HARD BUDGET
Return AT MOST <N> results. Open at most <M> documents. After roughly 25 tool calls, STOP
investigating and return immediately with what you have, marking any unverified evidence as
NONE. An early return with 2 solid results is a SUCCESS; grinding to exhaustion is a FAILURE
that returns nothing at all. Do not enumerate directories exhaustively. Do not read large
files in full — read heads and grep.
```

Three companion rules:

- **Cap the schema too** — bound result counts and add `under N chars` to every free-text field. Unbounded prose fields are a second stall vector.
- **Split, don't raise the cap.** When a cluster dies, re-run that slice as two or three *narrower* clusters rather than retrying the same shape with a bigger budget.
- **Never report a partial census as complete.** Name every slice that failed to return, in the manifest and in the reply. A sweep that silently covered 8 of 12 clusters is the actual failure mode, worse than the stall itself.

## Phase 2 — Batched interview (staged sittings, not one-repo-at-a-time)

This is the "per-repo mini-interviews, batched" model. Do NOT interview one repo, plan it, then
interview the next — audit ALL first (Phase 1), then run the questions as **staged sittings**:

- **Sitting vehicle follows the standing GitHub-sheet policy** (28-08-2026): a sitting is a collected session of ≥2 decisions, so each sitting becomes ONE interactive review-sheet published to `gasyoun.github.io/vote/` (the `/review-sheet` contract: approve/reject/defer per fork → downloadable `decisions.json`), handed over with the link + a 40-second orientation in chat. Apply the returned `decisions.json` via `/decisions-apply`, then write the rulings into the manifest. A single genuinely-mid-sitting question (1 item) may stay in chat.
- **When the human is actually sitting live in the session**, `AskUserQuestion` rounds (≤4 questions each) remain the vehicle; a single call may mix questions from **different repos** so one sitting clears many repos' load-bearing forks. Lead with priority-tier order — Tier-0 repos' forks first.
- Each option carries its repo tag, concrete trade-offs, and the recommended choice marked "(Recommended)".
- **Record every ruling to the manifest as it lands.** A sitting can end at any point — the manifest holds the rulings so a later `--resume` sitting continues with the un-asked forks. This is what makes the interview burden something you spread across sittings rather than one marathon.
- Any fork not worth your attention (one option clearly dominant) is NOT asked — it becomes a logged default in that repo's plan, entered in the decisions-taken table as `(default, not separately ruled)`, exactly as `/ask` does. Logged, never dangling.

## Phase 3 — Author the full plan per repo

For each repo whose interview is complete, author the full `/ask` Phase-3 five-doc set under its
`docs/`: `PLAN_<repo>_<subject>_<span>.md` (index + decisions-taken table + autonomy contract +
an explicit `Open @DECIDE` section stating `None.` when empty)
linking `ROADMAP_<repo>_<span>.md`, `ARCHITECTURE_<repo>_<subject>.md`,
`IMPLEMENTATION_<repo>_<subject>.md`, `VERIFICATION_<repo>_<subject>.md`. Doc contract applies
(dated header, byline, full blob URLs, no HTML). Register each roadmap in
[`ROADMAP_INDEX.md`](https://github.com/gasyoun/Uprava/blob/main/ROADMAP_INDEX.md) same pass;
give each PLAN index a sibling `.meta.md`. Tick the repo's plan-docs cell in the manifest.

## Phase 4 — Mint the tiered, QUEUED handoffs

Per repo, turn each wave-1 (and clearly-scoped later-wave) deliverable into a **queued** `H###`
handoff. **Model tier is auto-derived per task** and locked into the filename. Since H3688 (OxAlpha, any-lane drain unlock, 29-08-2026) the
filename token is **provenance, not a gate**: `next_task_scan.py` lets any lane pick any tier and
carries a filename/executor mismatch as `tier_mismatch` (a WARNING; `--strict-model-lock` restores
the old hard drop) — so a wrong token is never caught at launch, only by `/tier-audit` below:

- **Mechanical / deterministic** (batch conversion, registry sync, scripted census, markup fixes) → **Haiku 4.5 (claude-haiku-4-5-20251001) / Sonnet 5 (claude-sonnet-5)**.
- **Judgment / design / correctness-critical** (architecture, algorithm, review, correctness-sensitive build) → **Opus 4.8 (claude-opus-4-8)**.
- **Register-sensitive Russian / author-voice / editorial / strategy** → **Fable 5 (claude-fable-5)**.

Mint with `python Uprava/tools/mint_handoff.py --batch <specfile>` (**always `--batch` for a
series** — all N ids in ONE commit+push, one contention window; never hand-pick from the marker).
Spec lines are `Model<TAB>repo<TAB>slug<TAB>title<TAB>effort<TAB>spec` — the 5th field
(`trivial|medium|hard`, H2162 effort gate) and the 6th (mission text, a spec-file path, or
`STUB` + a 7th `--fill-by` ISO date, H3124 spec gate) are REQUIRED on every line; one ungated
line refuses the whole batch (exit 2 for a missing effort, exit 5 for a missing spec; never a partial mint), so fill them before minting, not after.
Each handoff: status **🟡 queued**, a literal `Read <full-path-to-PLAN-doc> and execute it.`
start line scoped to its deliverable, and its parked `@DECIDE`s (if any) called out inside.
Registry rows + counts + marker land **automatically inside the batch claim commit** (H1788 (Fable 5), mint auto-writes registry rows, 28-07-2026 — adopted after this skill's own 27-07 staging wave minted 34 files with ZERO rows;
the prose step existed and was skipped): never hand-add a row (duplication), just VERIFY the
mint printed no "stubs-only" WARN — if it did, the regen.yml self-heal step rows the orphans
on the next CI tick. Before publishing any start lines, run `/tier-audit` over the minted
batch — a mis-tier gets its filename token + Model line fixed in the same pass. Record the
per-repo count and tier split in the manifest.

## Phase 5 — Gate, wire, and record

- **Autonomy-readiness gate per repo** (`/ask` Phase 4) — mechanical, not self-attested: run `python ~/.claude/scripts/ask_gate_check.py --plan <PLAN index>` and record its exit code in the manifest's gate-verdict column; the human halves (no-rebuild-what-exists, contract coverage of plausible ambiguities) stay attested in session. A dangling *blocking* `@DECIDE` in a wave-1 path fails the gate — flag that repo in the manifest as "needs a follow-up sitting" rather than shipping a plan that stalls.
- Genuine human decisions that remain → `Uprava/GTD_NEXT_ACTIONS.md` as `@DECIDE`; deploy/human `@DO`s likewise. Run `/gtd-sync` at the end of a pass.
- Update each repo's `.ai_state.md` Next Steps to point at its PLAN doc.

## Phase 6 — Deliver

Commit/push per each repo's rules (config/Uprava direct; guarded repos via worktree; nothing to
`csl-orig`). Report the manifest link and a totals table: repos staged / plans authored /
handoffs minted by tier / repos flagged for a follow-up sitting / repos still pending. Give the
literal `--resume` line so the next session continues:

```
Read ~/.claude/commands/ask-batch.md and execute it. --resume
```

## Hard rules

- **Nothing executes.** Every handoff is minted 🟡 queued — this skill stages a backlog, it does not run it. Launching is a later, separate act (`/next-task`, `/go`, or pasting a start line).
- **Resumable or it's broken.** The manifest is updated in the same pass as each unit of work; a paused/crashed session must lose nothing. Default to `--slice` pacing (prove on a small slice before the full fan-out) rather than attempting all repos in one blind pass.
- **Interview batched, never skipped and never one-repo-serial.** Audit all, then ask in sittings; record rulings immediately. ≥2 decisions for an absent human vote on the published GitHub sheet, never chat (28-08 standing policy).
- **Tier is auto-derived and filename-locked**; a mis-tiered handoff is a real defect (since H3688 the scanner only flags the mismatch as `tier_mismatch` provenance, so `/tier-audit` is the catch). Registry registration is automatic at mint since H1788 (Fable 5) — verify (no stubs-only WARN), never re-add rows.
- **Scope guards are honored** — `--since`, `--repos`, `--tier`, `--slice` bound the spend; a batch with no bound is capped to Tier-0/1 by default and reports what it deferred.
- **Heavy fan-out is opt-in.** The read-only audits (Phase 1) and the plan-authoring/mint (Phases 3–4) can be orchestrated as a multi-agent `Workflow` for scale, but only when the user explicitly opts into that token spend; absent opt-in, run sequentially by `--slice`.
- **Every fan-out agent is budgeted, and every dead slice is named.** No audit/harvest/verify agent ships without the early-return budget in its prompt body (Phase 1); no report claims coverage it did not get. Cluster failures are reported by name, not averaged away.

_Dr. Mārcis Gasūns_
