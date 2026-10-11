---
name: "security-defect-lane"
description: 'One-defect-per-handoff security remediation from a parent findings doc - re-verify on main before code, tick severity buckets. Triggers: "security wave", "one defect per handoff". Sonnet 5 (claude-sonnet-5); Opus 5 (claude-opus-5) framework-upgrade waves.'

---

# security-defect-lane

Use this skill when the user asks to run the migrated source command `security-defect-lane`.

## Command Template


# /security-defect-lane [parent-findings-doc | wave | H###]

_Created: 12-08-2026 · Last updated: 26-08-2026_

Encodes the remediation lane that ran **29 times in six days** across the H2323–H2591 window without ever being written down — census candidate **C32** in [SKILL_CANDIDATES_CENSUS_H2323_H2591_12_08_2026.md](https://github.com/gasyoun/Uprava/blob/main/archive/SKILL_CANDIDATES_CENSUS_H2323_H2591_12_08_2026.md) (archived snapshot-series family per H3101), force-built on MG's instruction 12-08-2026 ahead of the sheet vote ([H2593](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H2593-Opus_claude-config_skill-build-c32-security-defect-lane_12.08.26.md)).

The shape is **not** Systema-specific — [H2572](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H2572-Sonnet_BookIndex_nanoid-codeql-safeurl-security-fixes_10.08.26.md) already ran it in BookIndex. That is why this is a skill and not a `/systema-case-feature` mode.

Argument: the parent findings doc (e.g. a `SECURITY_ROADMAP.md` bucket), a wave label (`W2`/`W3`/`W4`), or a single `H###` unit to execute.

## Why a lane and not "fix the security issues"

A findings doc with 17 numbered defects invites one enormous session that fixes some, half-fixes others, and leaves nobody able to say which. The lane makes each defect **its own closable unit with its own evidence**, so partial progress is legible and a re-run never re-litigates what already landed.

| | Lane | The thing it replaces |
|---|---|---|
| Unit of work | one numbered defect, one handoff, one PR | "the security pass" |
| Done | that defect's bucket ticked with a cite | vibes |
| Partial | 6 of 9 units ✅, 3 named and open | "mostly done" |
| Re-run cost | re-verify (cheap) | re-implement (expensive, risks regression) |

## Phase 0 — Find the parent, never invent one

The lane needs a **numbered** parent findings doc. Locate it before anything else:

- A dated findings handoff (the org's canonical case: [H071](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H071-Fable_Systema-Sanscriticum_systema_money_core_findings_03.07.26.md), Systema money-core, numbered `#7`…`#17`).
- A repo `docs/SECURITY_ROADMAP.md` with wave sections and checkboxes.
- Scanner output (CodeQL/Semgrep/Dependabot) — **number it into a doc first**; an alert list is not a parent, because alerts renumber and disappear.

If no numbered parent exists, **build it first** and stop there: a census pass (Phase 1) is a legitimate whole session. Fanning out against an unnumbered backlog produces handoffs nobody can map back to a finding.

Check prior art before adding a finding: `/prior-art`, plus the repo's own `FINDINGS.md` and closed security PRs. Re-reporting a fixed defect as new is the most common defect of this phase.

## Phase 1 — Census the bucket (read-only)

For the wave you are working, produce a table with **one row per numbered defect**:

| Column | Content |
|---|---|
| `#` | the parent's number — never renumber, never re-sort |
| Title | one line, the defect, not the fix |
| Severity | CRITICAL / HIGH / MEDIUM / LOW — the parent's own call unless demonstrably wrong |
| Claimed fix | PR/commit cite if someone says it is already fixed, else `none` |
| Test signal | the test name or code path that would prove it |
| Status | `open` · `claimed-fixed` · `verified-fixed` |

`claimed-fixed` is **not** `verified-fixed`. The whole lane turns on that distinction.

## Phase 2 — Fan out: one defect, one handoff

Mint one handoff per `open` or `claimed-fixed` row — **always `/handoff-mint --batch`**, since N are known at this moment (that is exactly the "N≥2 due NOW" case the `--batch` rule quantifies over).

Each unit handoff carries, and nothing more:

- **Scope:** the single `#`, named. `Fail = inventing new scope` is a stated failure mode, not a nicety — a unit that grows into "and also refactor the guard" stops being closable.
- **Re-verify first** (see Phase 3), with the claimed fix cite inline.
- **Acceptance:** defect fixed on main · PR/test cite · no re-open of the leak.
- **Evidence required:** PR URL · test name **+ its presence on `origin/main`** · fix cite · roadmap note if the unit owns a sub-bullet.
- **Fail =:** force-close without re-verify.
- Effort is nearly always `trivial` for a verify unit, `medium` for one that implements.

Then mint the two **closer** units, which depend on the children:

1. **Bucket tick** — flips the wave's severity-bucket line to `[x]` once every child is closed.
2. **Parent exit tick** — flips the parent wave checkbox and the process-suite line once the bucket is ticked.

Both are `trivial`, and both carry `Fail = tick without re-verify`. Splitting them is deliberate: the bucket can complete while the parent wave still owes another bucket.

**Umbrella fallback.** Also mint one umbrella handoff holding the `#`→`H###` mapping table, with an explicit either/or: *prefer the one-defect fan-out; if the fan-out is missing, run this as a single census and tick the bucket.* That is what makes the lane survive a session that finds the children never got minted.

## Phase 3 — Execute a unit: RE-VERIFY BEFORE YOU TOUCH CODE

This is the load-bearing rule of the whole skill.

```
1. Read the defect on origin/main — the code path, not the memory of it.
2. Run / locate the test signal.
3. Is it still broken?
     NO  -> do NOT re-implement. Confirm on main, record the cite,
            tick, close against the ORIGINAL fix PR (+ a small verify PR
            if a doc/test note is owed). Done.
     YES -> implement the minimal fix for THIS defect only, with a
            regression test, one PR.
```

Re-implementing an already-fixed defect is the expensive failure this ordering exists to prevent: it burns a session, and it risks a *regression in working code* while claiming to improve security. In the observed window a large share of units closed on the NO branch.

**Money- and access-touching units** additionally route through [`/money-pr-land`](https://github.com/gasyoun/claude-config/blob/main/commands/money-pr-land.md) — a security fix that changes who can pay, be paid, or get access is a money-contour change regardless of the label on the handoff.

**Watcher-afflicted repos** (currently Systema-Sanscriticum) land every write through [`/watcher-safe-commit`](https://github.com/gasyoun/claude-config/blob/main/commands/watcher-safe-commit.md).

## Phase 4 — Tick, with the cite attached

A tick without a cite is indistinguishable from a lie six weeks later. Every bucket line gets: the `#`, the PR URL, and the test name. Then:

- close the unit via [`/handoff-close`](https://github.com/gasyoun/claude-config/blob/main/commands/handoff-close.md);
- when all children are closed, run the bucket-tick unit; then the parent-exit unit;
- **never tick the parent while a child is open** — the exit tick is a claim about the whole bucket.

## Phase 5 — Wave escalation

Waves are ordered by *how much of the system the fix can break*, cheapest and most-contained first. Do not skip forward.

| Wave | Content | Gate before entering |
|---|---|---|
| **W1** | CRITICAL/HIGH access + revenue defects | — |
| **W2** | MEDIUM/LOW residual, one-defect fan-out, bucket + parent ticks | W1 bucket ticked |
| **W3** | Scanner gates: triage advisory findings, document every dismissed rule, then flip the job to **required**; Dependabot auto-merge on green | W2 ticked. **Fail = flipping the job required with an untriaged noise flood** — that reliably ends in someone disabling the check |
| **W4** | Framework/runtime upgrades (Laravel, PHP), dependency-posture review, deploy-surface review | W3 gate green. Upgrades are `hard`, get Opus 5 tier, and each gets its own handoff (blocker inventory first — plugin/package compatibility often gates the upgrade itself) |

W4 dependency posture = read `composer.lock` / `package-lock.json` for abandoned and risky packages, produce a **dated report with actions**; `Fail = empty report` and `Fail = silent ignore of a known CVE`.
W4 deploy surface = read the actual `deploy.sh` / `docker-compose.yml` / CI workflow and prove no secret is echoed; `Fail = skipping the read and asserting it from memory`.

## Phase 6 — Propagate

- **Data-integrity findings open a GitHub issue in the owning repo the same pass** (standing org rule) — a security defect that silently corrupts or leaks data is integrity, not just security.
- Reusable caveats → [`/findings-append`](https://github.com/gasyoun/claude-config/blob/main/commands/findings-append.md).
- Every result table → the roadmap doc + the tracking issue, per persist-tables.
- The wave's completion → `CHANGELOG.md` and [`/cut-release`](https://github.com/gasyoun/claude-config/blob/main/commands/cut-release.md).
- Sibling repos with the same defect class → queued port, named explicitly. (This is how the lane reached BookIndex.)

## Hard rules

- **Never implement before re-verifying on `origin/main`.**
- **One defect per handoff.** A unit that grows scope gets split, not widened.
- **Never tick without a cite**; never tick a parent over an open child.
- **Never flip a scanner gate to required over untriaged noise.**
- A unit that turns out already-fixed is a **successful** unit — closing it on the verify branch is the point, not a shortfall.
- Report partials honestly: `6/9 closed, 3 open and named` beats "security pass done".

## Not this skill

| Situation | Use instead |
|---|---|
| Org-wide CI/secrets/SAST posture across many repos | [`/cologne-security-audit-all`](https://github.com/gasyoun/claude-config/blob/main/commands/cologne-security-audit-all.md) |
| Reflected-XSS / injection hunt in PHP web code | [`/cologne-php-xss-sweep`](https://github.com/gasyoun/claude-config/blob/main/commands/cologne-php-xss-sweep.md) |
| One-shot review of the current diff | built-in `security-review` |
| A single Dependabot PR's risk call | [`/dependabot-pr-triage`](https://github.com/gasyoun/claude-config/blob/main/commands/dependabot-pr-triage.md) |
| Secret already committed | [`/secret-purge`](https://github.com/gasyoun/claude-config/blob/main/commands/secret-purge.md) |

_Dr. Mārcis Gasūns_
