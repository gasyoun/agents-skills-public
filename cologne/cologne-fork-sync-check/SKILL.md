---
name: "cologne-fork-sync-check"
description: 'Detect/reconcile drift between repos with hand-synced forked copies of the same files (csl-websanlexicon basicadjust.php) — diff vs last-known-synced commit, sync via their script. Sonnet 5 (claude-sonnet-5). Triggers: "any fork drift", "basicadjust sync". Do not use for re-vendoring the canonical sanskrit-util library — that is /cologne-sanskrit-util-sync; this only checks hand-synced forked file copies like basicadjust.php.'

---

# cologne-fork-sync-check

Use this skill when the user asks to run the migrated source command `cologne-fork-sync-check`.

## Command Template


# /cologne-fork-sync-check

_Created: 04-07-2026 · Last updated: 05-09-2026_

`csl-apidev` and `csl-websanlexicon` each keep their own copy of `basicadjust.php` and
`basicdisplay.php` (PHP display logic), synced by hand via `apidev_copy.sh`. This has
drifted repeatedly without detection — commits `86ecf7f`, `a87cac4`, `356e820`, `cb0bca9`
each re-synced after the fact, and three separate `.ai_state.md` commits
(`b0b55eb`, `9b5c6b6`, `18c1842`) exist ONLY to note "concurrent basicadjust.php edit not
touched" — i.e. a missed sync was caught by luck, not tooling. One commit (`fa50eb6`)
self-describes the arrangement as "spaghetti code." This skill is the missing detection
step: run it BEFORE assuming either repo's copy is current, not just when something
already looks broken.

## The one-command form (default)

The whole check is a committed, deterministic script —
[`Uprava/tools/cologne_fork_sync_check.py`](https://github.com/gasyoun/Uprava/blob/main/tools/cologne_fork_sync_check.py)
(the [`H844`](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H844-Opus_claude-config_cologne-fork-sync-check-to-python_13.07.26.md)
(Opus 4.8) claude-config cologne-fork-sync-check ported to Python; nothing here needs a model — detection is a blob compare, the only judgment
branch escalates to a **human**):

```sh
python ~/Documents/GitHub/Uprava/tools/cologne_fork_sync_check.py [--apply]
```

It `git fetch`es each repo and compares each side's **published** `origin/<default>` state (not
the local working tree — a stale local checkout must not masquerade as drift; `--no-fetch` for
offline). It reads the registered pairs from `apidev_copy.sh`'s own `cp` lines, byte/blob-compares
each side, and classifies drift **without a stored last-synced commit** by testing git blob-hash
presence across each side's history (Phases 1–2): one-side-moved → that side is authoritative;
neither current content in the other's history → both moved → a real conflict it refuses to
resolve and flags for a human. `--apply` copies authoritative→stale for the clear one-side-moved
cases and byte-verifies parity (Phase 3), printing the branch/PR + `.ai_state.md` record steps —
it never pushes a default branch and never silently picks a winner in a conflict. Exit 0 = clean
or cleanly-classified; exit 1 = a conflict/ambiguous pair needs a human. The phase docs below are
what the script does, and the manual fallback.

## Registered fork pairs

Maintain this list as the skill evolves — start with the known pair, add more if
discovered:

| File | Side A | Side B | Sync mechanism |
|---|---|---|---|
| `basicadjust.php` | `csl-apidev/.../basicadjust.php` | `csl-websanlexicon/v02/.../basicadjust.php` | `csl-websanlexicon/v02/apidev_copy.sh` |
| `basicdisplay.php` | `csl-apidev/.../basicdisplay.php` | `csl-websanlexicon/v02/.../basicdisplay.php` | same script |

(Locate exact current paths with `Glob` in both repos — they've moved before; don't
hardcode a path that might be stale.)

## Phase 1 — Diff both sides

For each registered pair: read both files' current content and their git blame /
last-modified commit + date on each side. Diff them directly (not just timestamps —
timestamps lie across repos with different commit cadences).

## Phase 2 — Classify

- **Identical** — nothing to do, record the check timestamp.
- **One side moved, other stale** — the moved side is authoritative; propagate via
  `apidev_copy.sh` (or direct copy if the script is missing/stale) to the other repo.
- **Both sides moved since last sync (real conflict)** — do NOT silently pick one. Show
  both diffs to the user; this needs a human call on which change wins or whether both
  need merging by hand. This is the case the three "not touched" commits were papering
  over — surface it, don't bury it in another docs-only commit.

## Phase 3 — Apply and verify

After propagating: byte-diff the two copies again to confirm exact parity (not just "ran
the script"). Commit on the repo that received the update
(`chore: sync {file} from {other-repo} ({date})`), branch, PR, auto-merge — never push
default branch directly.

## Phase 4 — Record

Note the check (clean / synced / conflict-flagged) in both repos' `.ai_state.md`. If a
real conflict was found and resolved by the user, log the ruling — this is exactly the
kind of decision that belongs in `Uprava/GTD_NEXT_ACTIONS.md` or a `/decision-record` if
it recurs with the same shape (e.g. "which repo owns basicadjust.php canonically" is
itself a decision worth locking once rather than re-litigating on each drift).

## Longer-term note

If drift keeps recurring, the durable fix is de-duplication (one canonical file,
consumed by both, per the `SHARED_CODE.md` dedup pattern already used for
transcoders/normalizers) rather than permanent hand-sync. Flag that option to the user
after the second or third real conflict this skill catches.

_Dr. Mārcis Gasūns_
