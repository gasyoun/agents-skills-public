---
name: "cologne-sanskrit-util-sync"
description: 'Re-vendor canonical sanskrit-util (Py+JS) into registered consumers after a version bump — detect stale copies, byte-diff verify, one PR per consumer. Haiku 4.5 (claude-haiku-4-5)/Sonnet 5 (claude-sonnet-5) tier. Trigger: "sync sanskrit-util consumers". Do not use for hand-synced forked file copies like basicadjust.php — that is /cologne-fork-sync-check; this re-vendors the canonical sanskrit-util library itself.'
---

# cologne-sanskrit-util-sync

Use this skill when the user asks to run the migrated source command `cologne-sanskrit-util-sync`.

## Command Template


# /cologne-sanskrit-util-sync

_Created: 04-07-2026 · Last updated: 05-09-2026_

## Default path (seconds — run this first)

```sh
python ~/Documents/GitHub/Uprava/tools/sync_sanskrit_util.py
python ~/Documents/GitHub/Uprava/tools/sync_sanskrit_util.py --apply
python ~/Documents/GitHub/Uprava/tools/sync_sanskrit_util.py --repo WhitneyRoots --apply
python ~/Documents/GitHub/Uprava/tools/sync_sanskrit_util.py --selftest
```

Script: [`Uprava/tools/sync_sanskrit_util.py`](https://github.com/gasyoun/Uprava/blob/main/tools/sync_sanskrit_util.py) (H3345 (OxAlpha), port of the H1967 detection core). Dry-run default = inventory of the registered vendored paths vs the sibling `sanskrit-util` clone + a per-consumer refresh plan — writes nothing. `--apply` executes the plan end-to-end: per consumer repo ONE `chore/re-vendor-sanskrit-util-vX.Y.Z` branch, wholesale copy sha256-verified before commit, `.js`-not-`.mjs` rename enforced, push + `gh pr create --auto --squash` — never a direct default-branch push; dirty consumer clones are refused. Detection table is single-sourced from [`cologne_sanskrit_util_sync.py`](https://github.com/gasyoun/Uprava/blob/main/tools/cologne_sanskrit_util_sync.py), whose own `--apply` remains the local-copy-only fallback when you deliberately want no PRs.

## After `--apply` (skill residue)

For each stale/missing consumer that was copied:

1. Smoke the consumer (build / load page for Observable — vendored as **`.js` not `.mjs`**).
2. Update [`SHARED_CODE.md`](https://github.com/gasyoun/github-spine/blob/main/SHARED_CODE.md) consumer rows with version + date + PR link (branch+PR+auto-merge is automated by `--apply`; grab the PR URLs from its output).

Queued "next consumer" migrations (replace local translit with vendored import) stay judgment — do not force when the package lacks the needed API; log the gap back to SHARED_CODE.

_Dr. Mārcis Gasūns_
