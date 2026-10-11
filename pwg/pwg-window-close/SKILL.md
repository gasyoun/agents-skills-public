---
name: "pwg-window-close"
description: Close ONE completed PWG→RU headless window safely — validate audit/provenance/schema/key delta vs manifest-v2 output, requeue --no-tm keeping better attempt, promote + TM rebuild, ledger row. After /pwg-bounded-run or mid-close session death. Do not use to start or gate a run — that is /pwg-live-gate or /pwg-bounded-run; this only closes out an already-completed window.
---

# pwg-window-close

Use this skill when the user asks to run the migrated source command `pwg-window-close`.

## Command Template


# /pwg-window-close

_Created: 12-07-2026 · Last updated: 06-09-2026_

Argument: a completed window's **coordinator lease / current-attempt artifacts + the headless manifest-v2 output** (or a `wf_output.json`-shaped result already on disk) + the `<root>`/window label. Optional: the lane (`verb` / `no_pwg` / `nominal` / `medium50`) for the ledger row. A Workflow **taskId**, if present, is **historical-forensics metadata only** — never the live capture source.

This is the **tail of the drain loop, factored out** — everything after a bounded window's calls return: capture → validate → requeue → promote → TM rebuild → ledger → commit. `/pwg-drain` runs Gate 0 / preflight / generate / **/pwg-bounded-run**, then hands the completed window here. Use it standalone to **resume a window whose session died mid-close** (the H148 lost-output failure mode) — the headless output + the coordinator current-attempt artifact are durable on disk; do not regenerate.

Working dir: `~/Documents/GitHub/SanskritLexicography/RussianTranslation`. Paths below are dual-platform: Windows spellings `src\pilot\...` = macOS/Linux `src/pilot/...` — same tree, forward slashes on darwin (`python3`). Generation tier is whatever produced the window (record the **resolved** version from the manifest, not a bare alias); this skill spends **no** generation tokens except during a requeue re-run.

Canonical mechanics (read only if a step is ambiguous, don't re-derive): [RUN_FREQ_MAX.md](https://github.com/gasyoun/SanskritLexicography/blob/master/RussianTranslation/src/pilot/RUN_FREQ_MAX.md) (worked example) · [PIPELINE_HISTORY.md](https://github.com/gasyoun/SanskritLexicography/blob/master/RussianTranslation/PIPELINE_HISTORY.md) (why each guard exists).

---

## ⚠️ Three guardrails — each cost a real loss to learn; never skip

These live here because they are **close-phase** rules; `/pwg-drain` links to them.

1. **Capture from the headless manifest-v2 output + coordinator current-attempt artifact — never a manual Max-surface save, never a truncated notice.** The headless CLI writes its full result to the `--output` file and its terminal state to the `--status-out` file; the coordinator holds the lease's current-attempt record. Read those directly. Do NOT trust any completion-notification text — it truncates. The H145 (Sonnet 5) `vid` run was lost exactly here: a manual save never reached disk and the stale file silently held the *previous* root (H148 (Fable 5) — max-workflow handoff gap audit; unrecoverable Max spend). A Workflow taskId is forensics-only; if one exists, its `.result` is comparison evidence, not the capture source.
2. **`--no-tm` is mandatory on every requeue-for-audit-failure round.** The TM is content-addressed on input SHA with no notion of "this cached translation already failed the gates", so a plain requeue re-serves the EXACT flagged content (the `gam` trap; the documented root cause of "half translations"). `requeue_from_audit.py` appends `--no-tm` automatically on anything but `--transient` — **verify it is on the generated harness** before running.
3. **"Latest requeue wins" is WRONG — a requeue round can regress a complete card.** `gam h0_63_sam_0` went 2→3→7 missing fragments across three retries. When a retry produces a *worse* card than a prior round, keep the better prior attempt; never overwrite a complete card with a fresh partial. Since H304, `save_and_audit --merge` enforces better-attempt-wins and the audit emits `requeue.defect.fshas.txt` so `requeue_from_audit` denylists the flagged fragments (guardrail 2 extended to the fragment TM) — verify, don't re-implement.

---

## Phase 1 — Capture (guardrail 1)

- If given the **coordinator lease / manifest-v2 output**: read the headless `--output` result (the complete `{meta,summary,results}`, uncapped) and confirm `meta.root` / the manifest's `selected_keys` match the `<root>` argument and the lease's current-attempt — a mismatch is exactly the H148 wrong-root loss. The `--status-out` file carries the terminal `stop_reason` (`AWAITING_REVIEW` for a `--stop-before-promote` window) and the manifest-v2 provenance (`profile_slot` · `config_dir_fingerprint` · `execution_route` · exact model).
- If given a **result JSON** already on disk: confirm `meta.root` matches before proceeding.
- **Reorder (hygiene, optional):** reorder `results` to `meta.selected_keys` order. `window_provenance.stale_check` compares as sets now, so this is cosmetic — but it keeps the captured result canonical/diffable.

## Phase 2 — Independently validate + classify

Do not trust the run's self-report; re-derive each invariant:

```sh
python3 src/pilot/audit_window.py <output.json> --root <root> --write-requeue
python3 src/pilot/classify_run.py <output.json>     # clean | code-failure | infra-confounded
```

Independently confirm, from the captured output + manifest, before promoting:
- **audit** clean (or acceptable partials) — `audit_window.py` writes status/report/ledger + `requeue.keys.txt` (split into `requeue.transient.keys.txt` = null cards and `requeue.defect.keys.txt` = gate-flagged) + `judge_sample.keys.txt`;
- **provenance** complete — exact model version, generator, rootmap_sha, input hashes, and the manifest-v2 `profile_slot` / `config_dir_fingerprint` / `execution_route` all present;
- **final schema** valid on every promoted card;
- **exact key delta** — the promoted key set equals `selected_keys` minus documented requeue/defer, no silent additions or drops;
- **store backup** — a fresh hash-addressed backup exists before any store write, and every pre-existing backup is preserved (delete none).

The harness `summary` returns `kill_timeouts` / `conn_errors` / `heal_calls` / `kill_bisect_blocked`; `classify_run.py` reads them and returns the verdict from the payload alone (hand-counting transcripts is over — H462):

- **`infra-confounded`** → **STOP. Do not tune budgets, do not burn requeue rounds** (H442 rule). The window is generation-host degradation noise (kill-timeouts at/near the `KILL_CEIL` — **600 s since 07-08-2026**, `HARD_TIMEOUT_MS` = `execution_contract.PRODUCTION_HARD_TIMEOUT_MS`; 300 s on 02-08→07-08 windows, 180 s before that — read the constant from code, don't trust this prose — regardless of skeleton size, conn-errors), not a code defect. Record the reading with `python src\pilot\probe_log.py outcome --run-id <run-id> --clean <n> --cards <m> ...` and leave the window unpromoted; re-run only after **/pwg-live-gate** returns GO.
- **`code-failure`** → a real gate defect; fix the code, don't paper over it with a requeue.
- **`clean`** (or acceptable partials) → continue to requeue/promote.

## Phase 3 — Requeue (guardrails 2 + 3)

If `requeue.keys.txt` is non-empty and the run is not `infra-confounded`:

- `python src\pilot\requeue_from_audit.py <root> --transient` for the cheap null-only re-run (state `transient_only`); `python src\pilot\requeue_from_audit.py <root>` for the defect set (**verify `--no-tm` is on the generated harness**).
- For a partial card with named `missing_fragments`, prefer `python src\pilot\autosplit_requeue.py topup` / `topup-merge` (re-translates only the failed fragment groups) over a full re-split.
- Re-run the window through **/pwg-bounded-run** (headless, not a Workflow session — guardrail 1), then re-enter Phase 1 on the fresh result — until requeue is empty **or clearly diminishing** (compare to `gam`: ~2 rounds cleared most; a stubborn few need a human). When a round regresses a complete card, keep the better prior attempt.
- Cap requeue at ~3 rounds (a 4th burns tokens on cards a human must resolve). This is the **G7** goal loop — hand it to an evaluator when the stop condition is worth automating. **Bootstrap mode forbids requeue entirely** (a `--stop-before-promote` bootstrap window parks at `AWAITING_REVIEW`; requeue is a later, non-bootstrap mode).

## Phase 4 — Promote + one-time TM rebuild

After gates PASS and a store backup exists:

```sh
python3 src/promote_final_cards.py --gen-model-version <exact-version> --glob "<tag>.<root>.json"   # NOT src/pilot/ — the promoter lives in src/
python3 src/pilot/translation_memory.py build --lang ru
python3 src/pilot/translation_memory.py build-frags --lang ru            # only if a heal emitted frag_prov
```

**Window-name footgun:** `promote_final_cards.py` ingests a **glob** of result files (`--glob`, default `DEFAULT_GLOB`). Scope `--glob` to **this root's own** files — an unscoped glob promotes every matching window on disk (or a stale one), the same wrong-root class as the H148 loss. Use `--dry-run` first to see which files it would read; `--merge` keeps the better prior attempt (guardrail 3). Resolve the model **alias to the exact version** from the manifest — a bare `sonnet` in provenance is a defect (`sonnet` resolved to `claude-sonnet-5` on 02-07, was 4.6 on 30-06). Never hand-append the store; `promote_final_cards.py` stamps provenance (model / generator / rootmap_sha / input hashes / generated_at) and writes `src\pwg_ru_translated.jsonl`. It **refuses synthetic/unknown provenance** (C-05) — a `synthetic_control` canary can never be promoted. Promoted rows are `review_status='ai_translated'`, **NOT approved** — `export_interop.approved_store()` still gates on `{approved,human_reviewed}` so unreviewed AI never reaches the citable edition. The TM rebuild is **one-time** per promotion, not per card.

> **⚠ TWO promote routes, DIFFERENT TM side effects (D6, H2173 G9) — this is why the TM commands above are spelled out rather than assumed.** A window can reach the store two ways, and only one rebuilds the TM for you:
>
> | Route | How it promotes | TM rebuild |
> |---|---|---|
> | **Coordinator** (`promote_ready`) | `coordinator.build_promotion_derived()`, under the **held** promote claim | **Automatic** — `translation_memory.build` + `build_frags` run inside it, and are re-run in a `finally`, so even a post-store-commit failure leaves the RU TM rebuilt (pinned by H1420 P10) |
> | **Manual `--glob`** (the block above) | `promote_final_cards.py` called directly | **NOT rebuilt** — the two `translation_memory.py` commands above are yours to run |
>
> So those commands are **required on the `--glob` path and redundant after a coordinator promote**. Running them twice is harmless (the build is idempotent); *skipping* them on the `--glob` path leaves the TM silently behind the store — which does not fail anything, it just spends agents re-translating what the TM already knew on the next window.

Then verify completeness + parity (the gate that would have caught "gam sat at 6/127 for weeks, invisible"):

```sh
python3 src/pilot/ru_coverage.py --min 100 --strict-denominator
python3 src/pilot/lang_parity_check.py
```

## Phase 5 — Ledger row + micro-commit

- Append **one row per promoted root** to the active lane's ledger, using the outcome legend (per-root outcome, NOT finding severity): 🔵 **clean** (gates PASS, zero residuals, TM rebuilt, provenance complete) · 🟡 **partial** (promoted with a documented residual or still mid-requeue) · 🔴 **error** (unresolved gate failure, requeue exhausted, or output lost — root stays queued). Row shape: `| <root> | <dot> | <agents> | <note: N/M clean, residuals, requeue rounds, promoted version, TM card/frag counts> |`.
- Append the window to `src\pilot\RUN_LOG.md` (per-run result + per-step model tier) and run `python src\pilot\check_launch_ledger.py` so the launch ledgers stay consistent.
- `git commit` the batch with an `ai-wip:` prefix (in a worktree — the SanskritLexicography main tree is commit-guarded; see the repo's CLAUDE.md).
- Record the launch outcome in the probe log so `LAUNCH_STATS.md`'s denominator stays honest: `python src\pilot\probe_log.py outcome --run-id <run-id> --clean <n> --cards <m> ...`, then `python src\pilot\probe_log.py render`.

## Phase 6 — Hand off to human review

Promoted cards are AI-translated, not approved. When a batch of roots is promoted and clean, assemble the G5/G6 human gate with **/pwg-review-packet** (read its file / follow it): it selects the `ai_translated` cards, renders the DE→RU review cards, and produces the interactive sheet whose `decisions.json` flips rows to `approved`. Do not hand-flip `review_status`. A **/pwg-bounded-run** bootstrap window that parked at `AWAITING_REVIEW` is likewise handed to an Opus 4.8 (`claude-opus-4-8`) review before any promotion — the `--stop-before-promote` checkpoint is the terminal state, not a pause to auto-resume.

## Change log

Newest first. This section exists so the metadata line at the top can stay two dates long.

| Date | Change |
|---|---|
| 06-09-2026 | H4201 darwin routes |

_Dr. Mārcis Gasūns_
