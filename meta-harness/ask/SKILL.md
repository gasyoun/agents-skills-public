---
name: "ask"
description: 'Front-load ALL human input into an up-front interview — live chat rounds, `--sheet` async review-sheet mode, or `--fast` one-sitting defaults lane — then author a full layered plan as cross-linked docs complete enough to drive a 5-8h autonomous build. Triggers: "plan this out fully", "ask everything now", "plan for an unattended build". Opus 4.8 (claude-opus-4-8)/Fable 5 (claude-fable-5) interviews.'

---

# ask

Use this skill when the user asks to run the migrated source command `ask`.

## Command Template


# /ask <repo-or-subject>

_Created: 13-07-2026 · Last updated: 05-09-2026_

The name is the method: **ask everything now, so the execution agent never has to ask.**
`/roadmap-interview` stops at a roadmap and leaves genuine forks as `@DECIDE`. `/ask` goes
the whole way down — it front-loads a heavy interview, resolves *every* fork a builder would
hit, and emits a layered plan (roadmap → architecture → implementation → verification → risks)
that a fresh agent can execute unattended for 5–8 hours. A plan that leaves an unresolved
blocking decision is a **defect** here: mid-build there is no human to answer it.

Composes `/roadmap-interview` (the roadmap layer) and the `Plan` agent (the implementation
layer); adds the up-front-interview discipline and an autonomy-readiness gate the others lack.

Argument: a repo name (resolve under `~/Documents/GitHub/`) or a subject/feature
within one. Optional flags: `--rounds N` (default: agent picks 3–5 by fork count),
`--sheet` (async review-sheet interview), `--fast` (one-sitting defaults lane — see Phase 2).

## Phase 1 — Audit before asking

Same discipline as `/roadmap-interview` Phase 1 — spend the human's attention only on genuine
forks, never on what evidence already answers. **Run `/prior-art` as part of this** (the
"check before you build" reflex): a plan that schedules rebuilding something a sibling repo
already owns is dead on arrival.

- `.ai_state.md`, `README`, existing roadmap/architecture/plan docs, `CHANGELOG` — what the repo says about itself.
- Open issues/PRs (`gh` — verify owner via `git remote -v` first; some repos are gasyoun personal, some org, some forks needing `--repo`).
- Git log since the last plan — what actually moved vs what was planned.
- The hubs — the repo's rows in [`Uprava/GTD_NEXT_ACTIONS.md`](https://github.com/gasyoun/Uprava/blob/main/GTD_NEXT_ACTIONS.md), [`Uprava/PROJECT_INTERLINKS.md`](https://github.com/gasyoun/Uprava/blob/main/PROJECT_INTERLINKS.md), [`SHARED_CODE.md`](https://github.com/gasyoun/github-spine/blob/main/SHARED_CODE.md), [`Uprava/REUSE_INDEX.md`](https://github.com/gasyoun/Uprava/blob/main/REUSE_INDEX.md) — who already owns the pieces this plan would build.

Produce a private shortlist: candidate workstreams, the target architecture's rough shape, and
the **full set** of decision points where two defensible options exist — priorities, arch forks,
tech/library choices, the acceptance bar, and the unattended-execution guardrails. Pre-fill a
recommended default for every fork as you go — `--fast` and residual defaults both consume it.

## Phase 2 — The up-front interview (ALL questions before any authoring)

This is the phase the skill is named for. Pick the mode by how the human can attend:

- **Default — live chat.** Elicit with `AskUserQuestion`, 3–5 rounds of **at most 4 questions
  per call**, all before a single doc is written. For a human actually sitting in the session.
  **4 is a hard tool cap, not a style preference:** `AskUserQuestion` rejects a 5th question with
  `InputValidationError … "questions": array too long (maximum: 4)`, and the round is lost. A
  round that genuinely needs 5–8 forks is TWO back-to-back calls of ≤4, not one oversized call —
  plan the split when you draft the round, not after the rejection.
- **`--sheet` — async.** For ≥2 decisions when the human cannot sit live: the standing
  GitHub-sheet policy bans chat vote-batches. Stage each round as ONE interactive
  review-sheet HTML published to `gasyoun.github.io/vote/` (the `/review-sheet` contract:
  approve/reject/defer per fork → downloadable `decisions.json`), send the link with a
  40-second orientation in chat, collect `decisions.json`, apply the rulings via
  `/decisions-apply`. Rounds still build in order — publish round k+1 only after round k's
  `decisions.json` has landed and been applied.
- **`--fast` — one sitting.** Opt-in only; the default method does not change. The audit's
  pre-filled defaults let the whole fork set resolve in ONE round: the human sees only the
  top ~6 genuinely load-bearing forks (the ones where a wrong default is expensive to undo);
  every other fork auto-locks to its logged default. Defaults land in the decisions-taken
  table marked `(default, not separately ruled)` — logged, never dangling, so the
  no-mid-build-stall guarantee holds.

Nothing is authored until the interview is complete — the point is to gather every ruling in
one sitting so the build never blocks. Each question: concrete options with trade-offs, your
recommended option FIRST and marked "(Recommended)". Never ask what the audit answers; never
re-litigate a locked decision.

Rounds build on each other — a natural progression (adapt to the repo, skip any round the audit
fully answers — Round 5 excepted, see the autonomy contract below):

1. **Goal & priorities** — what "done" means for this span, what to ship first, what's out of scope.
2. **Architecture forks** — the load-bearing structural choices: data model, component boundaries, interfaces/contracts, build-vs-reuse for each major piece.
3. **Implementation & tech** — libraries, patterns, file layout, migration/compat constraints, anything that fixes *how* wave-1 is built.
4. **Verification bar** — what proves each deliverable works, acceptance criteria, test strategy, the quality bar the autonomous agent must hold to.
5. **Unattended-execution guardrails** — the autonomy contract (see below): stop conditions, what to do on an unplanned ambiguity, commit/push/PR authority, what it must NOT touch.

**Crash-safe rulings (all modes).** After EACH round, append the rulings verbatim to
`docs/PLAN_<repo>_<subject>_DRAFT.md` — dated header, one row per ruling with rationale —
and update `.ai_state.md` Next Steps: `resume /ask <subject> at round k+1 — draft
docs/PLAN_..._DRAFT.md`. A session dying at round 3 loses nothing: resume = read the draft,
confirm the count with the human, continue at the next round. Phase 3 converts the draft into
the final PLAN index and the draft is then deleted — never leave an orphan DRAFT behind.

**The autonomy contract (Round 5 is non-optional).** Because no human is reachable for 5–8h,
the plan must pin the execution agent's behavior when reality diverges from the plan. Elicit and
record: (a) **on ambiguity** — pick the marked default and log it, or park-and-skip that item?
(b) **stop conditions** — what makes the agent halt vs press on. (c) **commit authority** —
per the handoff-scoped autonomy rule, wave-1 handoffs already authorize commit→PR→merge; confirm
any exception. (d) **the fence** — files/subsystems/data the agent must not touch. An agent with
no ambiguity policy either stalls or improvises destructively; this contract is what makes
"walk away for 8 hours" safe.

## Phase 3 — Author the layered plan (separate cross-linked docs)

Convert the DRAFT into the final docs, then delete the draft. One index + one doc per layer,
so each stays focused and the execution handoff can point at exactly the layer it needs. All
follow the doc contract: `DD-MM-YYYY` dated header (real git creation date if updating in
place), byline footer, full blob URLs with the correct org, no HTML. Self-identifying
filenames — subject tokens, not bare genre. Put them under `docs/` (or the repo's existing
plan path; update in place rather than spawning duplicates).

- **`docs/PLAN_<repo>_<subject>_<span>.md`** — the index/cover: one-paragraph goal, the decisions-taken table (every Phase-2 ruling + rationale, including `(default, not separately ruled)` rows), an explicit `Open @DECIDE` section (state `None.` when empty), the autonomy contract verbatim, and links to the four layer docs below. This is the doc the execution handoff's starter line points at.
- **`docs/ROADMAP_<repo>_<span>.md`** — waves/quarters with concrete deliverables, each stating what unblocks it; explicit non-goals list. (Reuse `/roadmap-interview` Phase 3's structure; a programme-level roadmap may be linked instead of restated.)
- **`docs/ARCHITECTURE_<repo>_<subject>.md`** — component boundaries, data model/schema, interfaces & contracts, how the pieces fit, the build-vs-reuse verdict per piece (citing the prior-art check).
- **`docs/IMPLEMENTATION_<repo>_<subject>.md`** — file-level, step-ordered build sequence for wave-1 (invoke the `Plan` agent to draft this), each step naming the files it touches and its dependency on prior steps.
- **`docs/VERIFICATION_<repo>_<subject>.md`** — acceptance criteria per deliverable, the exact command/flow that proves each works, and the risks & spikes register (unknowns, risky assumptions, anything needing a spike *before* committing to the arch).

Cross-link all five with full blob URLs. Register the roadmap in [`ROADMAP_INDEX.md`](https://github.com/gasyoun/Uprava/blob/main/ROADMAP_INDEX.md) and give the PLAN index a sibling `.meta.md` per the metadocs rule (it's a consulted-across-sessions spec).

## Phase 4 — Autonomy-readiness gate (mechanical, not self-attested)

Run the committed checker on the authored index:

```
python ~/.claude/scripts/ask_gate_check.py --plan docs/PLAN_<repo>_<subject>_<span>.md
```

It verifies mechanically: dated header + byline present; links to the
`ARCHITECTURE_`/`IMPLEMENTATION_`/`VERIFICATION_` layer docs (`ROADMAP_` missing = warning, a
programme-level link is legal); a rulings/decisions table exists; an autonomy-contract section
with a stop/ambiguity policy exists; and **zero unresolved forks** — the `Open @DECIDE`
section must say None, and no `@DECIDE` token may appear anywhere else in the doc.
`--emit` appends a dated "Autonomy gate" section with the verdict table to the PLAN doc, so
the gate leaves evidence instead of trust-me prose. Exit 0 = pass, 1 = fail.

The checker reads only the index; the human-judgment halves stay with you: confirm
**no rebuild-what-exists** (prior-art verdicts recorded, nothing scheduled duplicates an
existing asset) and that the **autonomy contract covers the plausible ambiguities** the audit
surfaced.

Any FAIL → the honest outcome is a *short* follow-up round (same mode as the interview), fix,
re-run. Never ship a plan that stalls at hour two. Say so plainly when the gate bounces.

## Phase 5 — Wire the autonomous execution

- Mint the wave-1 execution handoff(s) — `/handoff-mint` (`python Uprava/tools/mint_handoff.py <Model> <repo> <slug> --effort <trivial|medium|hard> --mission "..."` or `--spec-file <path>` — a bare mint with no spec is refused with exit 5; `--batch <specfile>` for several), each with a literal `Read <full-path-to-PLAN-doc> and execute it.` starter line in a fenced block. The handoff body points the execution agent at the PLAN index and names its scope; the index carries everything else.
- **Cost honesty:** next to each mint, name its expected cost class — the effort token plus a rough spend band. An 8h authorization should state what it costs before the human walks away. Then run `/tier-audit` over the batch to confirm tier×effort honesty before publishing starter lines.
- Human actions/decisions that legitimately remain (non-blocking, or post-build) → [`Uprava/GTD_NEXT_ACTIONS.md`](https://github.com/gasyoun/Uprava/blob/main/GTD_NEXT_ACTIONS.md).
- If `CHANGELOG` was touched, `/cut-release` same pass. Update `.ai_state.md` Next Steps to point at the PLAN doc.

## Phase 6 — Deliver

Commit + push per the repo's rules — for Uprava-targeted plans that means a worktree off
`origin/main` landed via `/land-on-main` (the shared main tree is guarded; never commit in
it), for other repos their own landing rules apply. The config repo itself is
guarded too → a worktree off `origin/main`, landed with `git push origin HEAD:main`. Report: the PLAN index link, the four
layer-doc links, rulings count (separately-ruled vs defaults-locked), the autonomy-gate
verdict (checker exit code + pass / short-follow-up-needed), and the execution handoff(s)
with their literal starter lines in fenced code blocks — so the human can paste one into a
fresh chat and walk away — and `elapsed: Xм Yс` for the run.

## Hard rules

- **Interview first, author nothing early.** All 3–5 rounds land before Phase 3. Front-loading is the whole method; interleaving questions with authoring recreates the mid-build stall this skill exists to kill.
- **Rulings persist per round.** The DRAFT file is written after every round, before the next is asked — a dead session must never cost ruled answers.
- **No blocking fork survives to execution.** Every decision is ruled or given a logged default. The Phase-4 checker run is mandatory; a plan shipped without its gate evidence is a defect.
- **≥2 questions for an absent human go on a sheet.** `--sheet` obeys the standing GitHub-sheet policy; live AskUserQuestion rounds are only for a human actually sitting in the session.
- **Guarded-tree preflight before the FIRST Write, not at deliver time.** Phase 3 authors real
  `.md` files, so the target checkout must be established before a single one is written — not
  discovered in Phase 6. Run once, at the top of Phase 3, for the repo the plan lands in:

  ```sh
  python ~/Documents/GitHub/Uprava/tools/checkout_freshness.py --repo <repo> || true
  git -C <repo-path> rev-parse --show-toplevel
  ```

  If the path is a guarded main-tree checkout, claim a worktree (`/worktree-claim`) and author
  into it. Authoring straight into a guarded tree is refused by `pretooluse_worktree_isolation_required.py`
  **after** the doc is already composed, and every drafted file has to be re-aimed by hand.
- **Prior-art checked, or the plan is an opinion.** Build-vs-reuse is decided with evidence for every major piece; a plan that schedules reinvention fails.
- **The autonomy contract is part of the plan**, not an afterthought — an unattended agent with no ambiguity/stop/fence policy is unsafe to launch.
- This is the heavyweight tool. For a quick single-repo roadmap with no long autonomous build behind it, use `/roadmap-interview`; for unattended planning where nobody can answer at all, `/roadmap-interview --no-interview`.
- **Many repos at once?** `/ask-batch` is the portfolio mode — it stages this same layered plan across every repo you worked in over a window and mints tens–hundreds of queued, tier-locked handoffs "ready when the time comes", batching the per-repo interviews into staged sittings and tracking everything in a resumable manifest. Reach for it instead of running `/ask` by hand across a list.

_Dr. Mārcis Gasūns_
