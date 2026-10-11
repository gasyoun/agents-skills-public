---
name: "account-switch"
description: 'Switch the local Claude Code CLI to another account (e.g. a Max plan that hit its weekly limit) and RESUME the exact mid-task session — same conversation, same context, no manual re-briefing. Triggers: "switch accounts", "weekly limit is exhausted".'

---

# account-switch

Use this skill when the user asks to run the migrated source command `account-switch`.

## Command Template


# /account-switch

_Created: 15-07-2026 · Last updated: 26-09-2026 (H5502: mechanical route ported)_

## Default path (seconds — run this first)

The gh-side mechanical bulk — account inventory, credential presence, the switch itself — is ported to a deterministic script (Uprava `tools/account_switch.py`, H5502):

```sh
# what the battery WOULD run (DEFAULT, zero subprocess):
python3 ~/Documents/GitHub/Uprava/tools/account_switch.py
# live read-only check — parse `gh auth status`, list accounts + active one:
python3 ~/Documents/GitHub/Uprava/tools/account_switch.py --check
# local credential battery (gh presence, git identity for a repo, token env PRESENCE as booleans):
python3 ~/Documents/GitHub/Uprava/tools/account_switch.py --battery --repo ~/Documents/GitHub/Uprava
# actually switch gh accounts (explicit flag required; unknown names refused):
python3 ~/Documents/GitHub/Uprava/tools/account_switch.py --account NAME
```

LLM / judgment residue only after the script report exists (H5502).

Gate that stays with the LLM: which account for the task — choosing WHICH identity a given piece of work must run under, and the Claude Code Keychain/credentials choreography (Phases 2–4 below, macOS Keychain included), are judgment and platform surgery the script deliberately does not touch.

Account identity lives in **one place per platform, and it is not the same place** -
this is the single fact to get right before running anything below:

| Host | Where the account token actually lives | How you move it aside |
|---|---|---|
| Windows | `%USERPROFILE%\.claude\.credentials.json` | copy/rename the file |
| Linux | `~/.claude/.credentials.json` | copy/rename the file |
| **macOS** | **the login Keychain**, generic password service `Claude Code-credentials` - **no `.credentials.json` exists on disk** | `security` (Phase 2/3 below), never a file copy |

Measured 03-09-2026 on darwin 24.6: `ls ~/.claude/.credentials.json` is `No such file or
directory` while `security find-generic-password -s "Claude Code-credentials"` returns the
item from `login.keychain-db`. A file-copy backup on a Mac therefore *silently backs up
nothing* and the later delete removes nothing - you reach Phase 4 still logged in as the
exhausted account, with no error to tell you why. **The conversation
itself lives elsewhere and is untouched by account identity**: every session
transcript is a local `.jsonl` file under
`~/.claude/projects/<project-path>/<session-id>.jsonl`, keyed by session ID,
not by account. `CLAUDE.md`, memory under `~/.claude/projects/*/memory/`,
hooks, skills, settings, and every repo's git state are equally
filesystem-local, not account-scoped. Switching credentials never touches
any of it — which is *why* resuming the same session under a different
account just works: the harness reads the transcript off disk regardless of
which OAuth identity is currently active.

**Zero-quota fact:** logging out and logging back in are harness-level CLI
actions, not model calls. They work even at 0% remaining weekly usage. Only
asking Claude to *run this skill for you conversationally* costs a turn — if
usage is down to a sliver, prefer running Phase 2–4 by hand in the terminal
over invoking `/account-switch` and hoping a reply completes.

## Phase 1 — Protect in-flight work (needs a working session; skip if already frozen)

1. `git status` in every repo with uncommitted changes from this session.
2. Commit anything not yet committed — once quota
   hits zero mid-response, there is no guaranteed chance to do this after.
   Stash only on private, single-session trees (`git stash` WITHOUT `-u`):
   in a shared main tree `git stash -u` captures ANOTHER session's
   uncommitted work (H859), so prefer a worktree commit there.
3. No need to hand-transcribe conversation context — Phase 5 resumes the
   actual session transcript, not a fresh one. The only things Phase 1 must
   catch are side effects that live *outside* the transcript: uncommitted
   files (step 2) and anything the current turn was about to do but hadn't
   yet (e.g. a pending tool call) — a half-run mutation is the one thing
   resuming the transcript can't retroactively finish safely; note it if so.

## Phase 2 — Back up current credentials (always do this, never skip)

Windows (PowerShell):

```powershell
Copy-Item "$env:USERPROFILE\.claude\.credentials.json" "$env:USERPROFILE\.claude\.credentials.<old-account-label>.bak.json"
```

Linux (sh):

```sh
cp ~/.claude/.credentials.json ~/.claude/.credentials.<old-account-label>.bak.json
```

macOS (sh) — there is no file to copy; export the Keychain item's secret to a
file you control, then verify it is non-empty before going any further:

```sh
security find-generic-password -s "Claude Code-credentials" -w \
  > ~/.claude/.credentials.<old-account-label>.bak.json
test -s ~/.claude/.credentials.<old-account-label>.bak.json && echo BACKUP-OK
```

Never delete the credentials outright — move them aside so the first account
can be restored later without a fresh OAuth flow. On macOS that backup file is
the *only* copy once Phase 3 removes the Keychain item, so `BACKUP-OK` is a
gate, not a nicety.

## Phase 3 — Log out

In a running `claude` session: `/logout`.

If the session is already unresponsive (0% quota, no reply coming back),
skip straight to removing the live credentials instead:

```powershell
Remove-Item "$env:USERPROFILE\.claude\.credentials.json"
```

```sh
rm ~/.claude/.credentials.json                                   # Linux
security delete-generic-password -s "Claude Code-credentials"    # macOS
```

(The Phase 2 backup already preserved them. On macOS `rm` on a path that does
not exist is the trap: it exits non-zero and looks like a permissions problem,
when in fact the credentials were never in the filesystem at all.)

## Phase 4 — Log into the second account

Run `claude` again — with no credentials present (file or Keychain item) it starts a fresh
OAuth login flow. Sign in with the second Max account.

## Phase 5 — Resume the SAME session (this is the point of the skill)

From the same working directory as the frozen session:

```powershell
claude --resume             # interactive picker, scoped to this directory — DEFAULT CHOICE
claude --resume "<keyword>" # picker pre-filtered by a search term
claude --resume <session-id>   # resume a specific session by ID once known
claude --continue           # shortcut: resumes ONLY the single most-recent session — see caveat
```

**Default to `--resume` (the picker), not `--continue`, whenever more than
one session could plausibly be "the one."** `--continue` silently grabs
whichever session in this directory was touched most recently — if *any*
other session (even an unrelated one-off question) ran in this directory
after the freeze and before the switch, `--continue` resumes the wrong one
with no warning. The picker lists every session with its start time (and
usually a first-message preview) so you can positively identify the frozen
one instead of trusting recency.

If the picker's previews aren't distinctive enough with many candidates,
disambiguate by file mtime — the frozen session's `.jsonl` stopped growing
at the exact freeze moment:

```powershell
Get-ChildItem "$env:USERPROFILE\.claude\projects\<project-path>\*.jsonl" -File | Sort-Object LastWriteTime -Descending | Select-Object -First 10
```

```sh
ls -lt ~/.claude/projects/<project-path>/*.jsonl | head -10      # macOS / Linux
```

then `claude --resume <session-id>` with the ID read off the matching
filename.

This reopens the actual `.jsonl` transcript — full conversation history,
not a summary, not a re-brief. No manual context restatement needed; that
was only ever a Phase-1 fallback for the rare half-finished side effect.

Confirm continuity:

- The resumed session shows the exact point work was cut off.
- `CLAUDE.md` and memory (`~/.claude/projects/.../memory/MEMORY.md`) load
  identically — they're filesystem-local, not account-scoped.
- Hooks and skills under `~/.claude/hooks` / `~/.claude/commands` are
  unchanged.
- If the very last turn was mid-stream when quota hit zero, sanity-check it
  (the tail of the transcript can be a truncated assistant message or a
  tool call with no result) before trusting it as ground truth — ask Claude
  to confirm what it last completed rather than assuming the cutoff was clean.

## Switching back later

Reverse Phase 2–4: back up the second account's credentials, restore the
first account's `.bak.json` copy, and repeat. On Windows/Linux the restore is
a file copy back over `.credentials.json`; on macOS it is a Keychain write,
because that is where the token lives:

```sh
security add-generic-password -U -a "$USER" -s "Claude Code-credentials" \
  -w "$(cat ~/.claude/.credentials.<old-account-label>.bak.json)"
```

If the backup was discarded, re-run `/logout` + `/login` instead. No data is
ever destroyed by this flow, only the active OAuth identity changes.

_Dr. Mārcis Gasūns_
