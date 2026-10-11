
The OpenCode session store bloats because the `event` table (an append-only streaming log
that re-emits whole message parts, incl. multi-MB git diffs) grows without bound. Real chat
content is `session`/`message`/`part`. The swap copies everything except `event` rows into a
fresh db and swaps it in. Proven results: 107 → 12 GB (Windows, 08-09, older prune method),
115 → 4.13 GB (Windows, 11-09), 82 → 2.6 GB (Mac, 15-09). **It regrows**: the Windows db went
from ~4 GB after its swap to 9.2 GB by 15-09; the Mac db went from 2.6 GB (15-09) to **45.9 GB
by 24-09 — ~5 GB/day** under the headless drain load, with the non-`event` payload only
4.3 GB. On a busy box the swap is a recurring chore (roughly weekly), not a one-off rescue.

**When it is worth it:** run the dry-run — it prints the copy size and what a real swap would
free. 15-09-2026: Mac 2.9 → 2.6 GB (not worth it), Windows 9.2 → 4.8 GB (marginal). Swap when
the dry-run shows more than ~5 GB or the drive is critically full.
**MG 17-09-2026: «opencode.db under 10 GB is nothing to be compressed»** — in the 17-09 rescue
(db 12.4 GB, dry-run projecting 7.5 GB freed) the swap was ruled off even with the drive
squeezed: Tier 1 had already restored ~25 GB, which made the swap redundant. Read the
bother-line as sitting well above the old ~5 GB guidance, and cite this ruling before
proposing a swap.

1. **Rehearse while OpenCode runs** (reads only; the agent may do this):
   `python3 ~/.claude/scripts/ssd/swap_opencode_db.py --dry-run` — on a full Windows C:, add
   `--staging D:\tmp_vacuum` (any writable dir on a drive with room). The dry-run writes the
   chat catalogue, builds the copy from ONE consistent snapshot, checks schema + row parity +
   `event` = 0, and runs the create-session write probe on the copy. Nothing is installed.
   The rehearsal copy is deleted at the end (since 24-09-2026; `--keep-copy` keeps it) — the
   real swap rebuilds it fresh, and a kept 4.3 GB copy once ate a third of the space Tier 1
   had just freed. It still needs roughly the copy size free while it runs.
2. **The owner runs the real swap** — it needs OpenCode fully closed, so it cannot run from
   inside an OpenCode session. Quit the desktop app AND any headless `opencode run` drain
   agents (they keep the db open after the app quits; drains are git-resumeable), then from a
   plain terminal run the same command without `--dry-run`. It refuses if anything holds the
   db (macOS: `lsof`; Windows: rename probe + `tasklist`) and prints who. The holder check
   runs only at the start: **keep OpenCode closed for the whole run (~10–20 min)**. On
   24-09-2026 the desktop app was relaunched 36 s before the copy finished (macOS logged a
   plain app launch, not the watchdog LaunchAgent); parity caught 2 new messages and aborted
   safely. An abort now deletes its partial copy and names whoever holds the db at abort
   time. Just re-run once `pgrep -fl OpenCode` prints nothing.
3. **The first OpenCode session after the swap is the check** (MG 15-09-2026): old chats
   render and a new message goes through → proven. The owner then deletes
   `opencode.db.bloated.bak`. The driver never deletes it, and refuses a second swap while it exists.
4. **If anything fails** after the original was moved aside (Ctrl-C and a move that fails
   halfway included), the driver restores it automatically. It puts back exactly the files
   it moved and never deletes an original it did not touch. One exception stops loudly
   instead: if the Windows holder check cannot rename the db back, it exits with CRITICAL
   and names the file to rename by hand. Manual restore: quit OpenCode, delete `opencode.db`
   (+`-wal`/`-shm`), rename `opencode.db.bloated.bak` back.
5. **Residual row:** a pending swap or `.bak` is a GTD `@DO` row the same pass — card ≤600
   chars (the GTD hook caps cards), narrative in the linked note.

Rules the kit encodes (each was a real bug) — do not re-derive them in a hand-written script:
stream rows (`fetchmany`, never `fetchall`/`OFFSET`); keep ALL DDL and skip only `event`
ROWS (a copy without the `event` table reads fine but every new session dies with
`no such table: event` — MSI 12-09); read every table inside one transaction (a live copy
without it got 66 orphan parts — Windows 15-09); probe the write path, because "reads work"
never proves "creates work" (H4612); move the `-wal`/`-shm` with the db so the `.bak` stays
consistent.

**Chat catalogue** — `CHAT_INVENTORY_<timestamp>.md` beside the db, written automatically at
step [2]: every session with directory, dates, message count, first user task and last
assistant result (MG ruling 11-09-2026: never ask whether to build it). **LOCAL-ONLY** — the
db carries `credential`/`account` tables; never commit the catalogue, never paste it into chat.

**Old box-local kits** (`swap_pruned_db.sh`, `swap_pruned_db*.cmd`, copies of the kit scripts
next to the db) are superseded by `~/.claude/scripts/ssd/`. A stale kit whose script blind-renames
an old staged copy silently drops newer sessions (11-09-2026) — rename such a script to
`.QUARANTINED-<date>`, never run it, delete its stale payload.

**Fallback without a second drive and without headroom** — prune + `VACUUM INTO` (proven
08-09, 107 → 12 GB): delete old `event` rows in chunks with `PRAGMA wal_checkpoint(TRUNCATE)`
after each commit (one big DELETE can grow the WAL past the remaining free space), then
`VACUUM INTO` a new file from a normal (not read-only) connection. Near 0 free, big scans die
with `database or disk is full` (temp spill) — free Tier-1 caches first.

