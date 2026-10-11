---
name: "ssd"
description: 'Disk-space rescue for a full drive on macOS or Windows — one read-only census script (free space + every known offender, sized and tiered), propose tiers, execute ONLY approved ones, verify the GB delta; tested swap kit for a bloated opencode.db. Never delete without approval. Triggers: "/ssd", "C: is full", "disk full", "clean space", "free up space", "1 gb left", "out of disk space", "диск забит".'
---

# ssd

Use this skill when the user asks to run the migrated source command `ssd`.

## Command Template


# /ssd [mac | drive | path]

_Created: 25-08-2026 · Last updated: 06-10-2026_

Interactive disk rescue: **measure → propose → execute what was approved → verify.** Never
delete anything the owner has not approved in this session. The same procedure covers macOS
and Windows; where the two differ, each phase says so side by side.

**The kit** lives in claude-config [`scripts/ssd/`](https://github.com/gasyoun/claude-config/tree/main/scripts/ssd),
and `~/.claude` is a claude-config clone on both boxes, so the path is the same everywhere:
`~/.claude/scripts/ssd/` (Windows: `%USERPROFILE%\.claude\scripts\ssd\`). Python 3 is all it needs.

| Script | What it does | Writes? |
|---|---|---|
| `ssd_measure.py` | Free space per volume + every known offender on this box, sized, tiered, sorted. `--drill PATH` = largest children of any dir | read-only |
| `swap_opencode_db.py` | The opencode.db copy-exclude swap, both OSes (§opencode.db rescue). `--dry-run` = rehearsal, reads the live db only | only at install |
| `compact_opencode_db.py` · `probe_new_session.py` · `chat_inventory.py` | Swap stages, called by the driver; each runs standalone too | copy / none / catalogue |
| `test_swap_opencode_db.py` | Acceptance bench: 18 checks on a fixture db (+ `--real-db PATH` on a copy) | temp dir only |
| `prune_opencode_events.py` | Weekly ONLINE cap for the opencode.db `event` table (H5644): deletes the oldest rows by rowid in committed chunks until the newest `--keep` 50000 remain, then `wal_checkpoint(TRUNCATE)`. `--dry-run` = read-only report; missing db exits 2. launchd runs it Sundays 05:17 on the Mac (`scripts/mac/com.gasyoun.opencode-event-prune.plist`); bench `test_prune_opencode_events.py` = 9 checks on a 1200-row fixture | deletes old event rows |
| `disk_watch.sh` | Hourly `df` of the local APFS Data volume; osascript alert once at ≤ `SSD_THRESH` GB free (default 25), clears above +5 GB, hysteresis band stays quiet; state-file dedup + bounded log, `--force-alert` pipe check (vps91_disk_watch.sh skeleton). MG 06-10-2026 option «c»: the 06-10 incident went 5 → 2 GB free inside one hour unseen. launchd runs it hourly at :41 (`scripts/mac/com.gasyoun.ssd-disk-watch.plist`) | alert only |

## Phase 0+1 — Measure (read-only)

```bash
python3 ~/.claude/scripts/ssd/ssd_measure.py
```

Windows (PowerShell or git-bash), the same script:

```bash
python %USERPROFILE%\.claude\scripts\ssd\ssd_measure.py
```

It prints free space for every volume (Windows: every fixed drive with a writability probe of
its root; macOS: `/System/Volumes/Data`, not `/` — the APFS system volume lies) and every known
offender that exists, largest first. Measured 15-09-2026: 33 s on the Mac, 64 s on the Windows
box, no hangs. A path that exceeds the per-path time budget prints as `>=N` (a lower bound).
Record the free-GB line as the baseline — the final report quotes the delta against it.

1. **Worktrees first on the Mac.** `drain worktrees` is usually the #1 row (38 GB in 81 dirs,
   15-09-2026). Route it to `/worktree-gc`, never to this skill's delete step. **The row size
   is not the yield**: 24-09-2026 it read 65.7 GB / 193 dirs, but `worktree_gc.py` found 7
   SAFE-REMOVE (2.2 GB) against 90 AT-RISK (dirty/unpushed — untouchable) and 154
   metadata-only ERROR rows (dir already gone, 0 bytes). Quote the SAFE-REMOVE sum when you
   propose it, not the census row. A full-estate `worktree_gc.py --skip-pr` run (160 repos)
   takes >10 min: start it in the background right after the census, before the ask.
2. **Unknown home dirs**: `--drill ~` after the census; anything big the census does not list
   is looked up (grep `commands/` for the dir name) BEFORE it can be proposed. 24-09-2026 found
   `~/.worktree-attic` (6.5 GB, human-only) and `~/.rescue-relay-*` (13.8 GB, never) this way —
   both are now census rows.
3. **Drill winners** with `--drill <path>` (1 level, sized, sorted) instead of ad-hoc `du`.
   A drive that is not the system drive (`/ssd D:`): `--drill D:\` gives the generic census.
4. **In-session trigger check**: if the disk filled up DURING this session, measure whatever
   was just installed or cloned first — it explains the trigger and bounds expectations
   (11-09-2026: a fresh 3.2 GB clone was the last straw on top of a 115 GB opencode.db).
5. **Staging drive** (Windows): a rescue that needs a big temporary copy picks the largest
   drive that is not full. Roots are often ACCESS DENIED while a subdir is writable — the
   census prints `root NOT writable (try a subdir)`; touch-probe the subdir before relying on it.
6. **Caches regrow overnight** — uv/huggingface grew +10 GB and worktrees 19 → 36 GB between
   14-09 and 15-09. Re-measure right before any big operation when free space is under ~10 GB.
7. **Windows full-disk masquerade (17-09-2026):** at 0 bytes free, failures surface as
   misleading network errors — a PowerShell `irm | iex` installer printed «failed to download
   from https://…» and `unzip` truncated with `write error (disk full?)`, while the asset was
   fine (HTTP 200; sha256 matched once space existed). Probe free space (or run this census)
   BEFORE blaming the link: a "download failed" that leaves a 0-byte/partial file is a
   disk-full symptom. Same family, new symptom (01-10-2026 MSI): a saturated C: silently
   disables crash dumps — volmgr Event 161 («Не удалось создать файл дампа») at every boot is a
   free-space symptom, not a storage fault; SMART verdict:
   [Uprava/reports/MSI_NVME_SSD_HEALTH_01-10-2026.md](https://github.com/gasyoun/Uprava/blob/main/reports/MSI_NVME_SSD_HEALTH_01-10-2026.md).

**No Python?** macOS: `du -xsh ~/.cache/* ~/Library/Caches/* 2>/dev/null | sort -rh | head -15`.
Windows: never `Get-ChildItem -Recurse` (hangs on this profile; junction loops like
`Application Data` recurse forever). Use robocopy list mode, run through cmd because git-bash
mangles `/L /E`: `cmd //c "robocopy <home>\<DIR> C:\__null__ /L /E /BYTES /XJ /NFL /NDL /NJH /NP /R:0 /W:0"`.
Parse the summary row by shape (`\S+ : <digits> <digits> 0 0 0 0`), not by the word
`Bytes` — Russian Windows prints `Байт`. Any PowerShell with `$_` goes in a `.ps1` file run
with `powershell -NoProfile -ExecutionPolicy Bypass -File` — inline `-Command` from git-bash
mangles `$_`.

## Phase 2 — Known offenders

The list inside [`ssd_measure.py`](https://github.com/gasyoun/claude-config/blob/main/scripts/ssd/ssd_measure.py)
is the source of truth; this table mirrors it. When a live run meets a new offender, add it to
the script first, then here. Tiers: **1** safe (re-downloadable) · **2** ask first ·
**H** human only · **N** never delete.

| What | macOS | Windows | Tier Mac/Win | Action |
|---|---|---|---|---|
| uv cache | `~/.cache/uv` | `%LOCALAPPDATA%\uv\cache` | 1 / 1 | delete the dir — `uv cache clean` hung >3 min on 4.2 GB (Mac, 10-09); on APFS expect well under the `du` size back — uv clone-links into `.venv`s (24-09: 23.4 GB `du`, Tier 1 total 34.9 GB `du` → ~15 GB `df`) |
| Hugging Face | `~/.cache/huggingface` | `%USERPROFILE%\.cache\huggingface` | 1 / 1 | delete; models re-download |
| npm | `~/.npm` | `%LOCALAPPDATA%\npm-cache` | 1 / 1 | `npm cache clean --force` |
| pip | `~/Library/Caches/pip` | `%LOCALAPPDATA%\pip\cache` | 1 / 1 | delete the dir — `pip cache purge` misses the macOS dir |
| Playwright / Puppeteer | `~/Library/Caches/ms-playwright`, `~/.cache/puppeteer` | `%LOCALAPPDATA%\ms-playwright`, `%USERPROFILE%\.cache\puppeteer` | 1 / 1 | delete; `npx playwright install` restores |
| pnpm store | `~/Library/pnpm/store` | `%LOCALAPPDATA%\pnpm\store` | 1 / 1 | `pnpm store prune` (low yield right after an install) |
| Homebrew, Xcode DerivedData | `~/Library/Caches/Homebrew`, `~/Library/Developer/Xcode/DerivedData` | — | 1 | `brew cleanup --prune=all`; delete DerivedData |
| Temp dir | `$TMPDIR` (7.5 GB, 15-09) | `%TEMP%` (18.5 GB, 15-09) | 1 / 1 | items >24 h old only; **always keep the `opencode` and `claude` subdirs** (live harness temp + Claude Code task outputs) |
| Stray opencode test dbs | `$TMPDIR`/`oc_*.db` | `%TEMP%\oc_*.db` | 1 / 1 | delete — probe/wal-test copies of opencode.db (16.7 GB pair 17-09-2026); producer NOT in kit code — if a kit stage starts emitting named dbs, teach it to clean up |
| Claude Desktop VM bundles | `~/Library/Application Support/Claude/vm_bundles` (11 GB, 15-09) | `%APPDATA%\Claude\vm_bundles` (~9 GB) | 2 / 1 | delete; re-downloads on next VM launch. Proven on Windows 08-09; on the Mac quit Claude Desktop first — it may be hosting the session |
| Codex runtimes | `~/.cache/codex-runtimes` | `%USERPROFILE%\.cache\codex-runtimes` | 2 / 1 | delete when no Codex session is live |
| App updater leftovers | `~/Library/Caches/com.openai.codex`, `…/com.anthropic.claudefordesktop.ShipIt`, `…/@opencode-aidesktop-updater` | `%LOCALAPPDATA%\routercheap\claude-desktop-backups` (~5 GB) | 2 / 2 | delete (Mac class MG-approved 10-09); Windows: list snapshot dates, drop the old ones |
| OpenCode tool cache | `~/.cache/opencode` | `%USERPROFILE%\.cache\opencode` | 2 / 2 | only with OpenCode closed |
| Ollama, Docker, iOS simulators | `~/.ollama`, `~/Library/Containers/com.docker.docker`, `~/Library/Developer/CoreSimulator` | `%USERPROFILE%\.ollama`, `%LOCALAPPDATA%\Docker\wsl` | 2 / 2 | `ollama rm <model>` per model the owner names; `docker system prune`; `xcrun simctl delete unavailable` |
| tgrep indexes, OpenCode undo snapshots | `~/.tgrep-indexes` (8.6 GB), `~/.local/share/opencode/snapshot` (2.4 GB) | same under `%USERPROFILE%` | 2 / 2 | tgrep: rebuildable, keep only served repos ([tgrep rule](https://github.com/gasyoun/claude-config/blob/main/rules/tgrep-fast-search.md)); snapshots: only with OpenCode closed |
| Downloads, Telegram, grok sessions | `~/Downloads`, `~/.grok/sessions` | `%USERPROFILE%\Downloads` (+ `Telegram Desktop`), `%USERPROFILE%\.grok\sessions` | 2 / 2 | size-ranked list via `--drill`; the owner picks per file (08-09: MG chose «show me the list first») |
| Drain worktrees | `~/Documents/GitHub/*-h<id>*` | same under `%USERPROFILE%` | 2 / 2 | `/worktree-gc` only — never `rm -rf` a git tree; never the current session's own worktree |
| Worktree attic | `~/.worktree-attic` (6.5 GB, 24-09) | same | H / H | salvage patches, secret-bearing, sometimes the only copy of commits (H2710) — no retirement rule; the owner decides ([/tip-rescue](https://github.com/gasyoun/claude-config/blob/main/commands/tip-rescue.md)) |
| Trash, hibernation | `~/.Trash` | `C:\hiberfil.sys` (6.3 GB) | H / H | the owner empties the Trash; `powercfg /h off` (admin, kills fast startup) is the owner's call |
| opencode.db (+`-wal`) | `~/.local/share/opencode/opencode.db` | `%USERPROFILE%\.local\share\opencode\opencode.db` | N / N | ALL OpenCode chats — never delete; bloat → §opencode.db rescue |
| Rescue relays | `~/.rescue-relay-*` (13.8 GB, 24-09) | same | N / N | bare git relay repos from `/tip-rescue` — rescue payload |
| whisper, page file | `~/.whisper` (daily dictation) | `C:\pagefile.sys` (34.6 GB, system-managed) | N / N | leave |

**Never propose at all:** `Documents/GitHub` repos (bloat there → `/worktree-gc`), npm globals
(`%APPDATA%\npm`), `%LOCALAPPDATA%\Packages` (Windows Store), browser profiles.

## Phase 3 — Propose tiers, WAIT for approval

Show the census rows as a table (path · size · what it is · tier), then ask with the question
tool (Claude Code: `AskUserQuestion`; OpenCode: `question`), one card per tier, recommended option first:

1. **Tier 1** — approve as a group. Nothing of value is lost. **STANDING APPROVAL (MG 06-10-2026, after the 2-GB incident): tier-1 rows carry a class approval — delete them without the per-session ask** and report them in the Phase-4 freed table; tiers 2/H/N still ask.
2. **Tier 2** — per item or as a group. `Downloads` always as a size-ranked file list.
3. **Human-only rows** — say what the owner would do and what it frees; the agent does not do it.

**Disk-full kill rule (MG 06-10-2026): «when ssd is full, cancel all freezes and kill all stale
worktrees without delay, always».** When free space is critical, the sentinel-breaker freeze on
the worktree-gc lane is cancelled by hand (delete `freeze.json` + a dated witness row in
`alarms.log`, per the H5182 silent-thaw lesson) and stale worktrees go via
`worktree_gc.py --apply` with the sanctioned env budget `WORKTREE_GC_MAX_HOUR`/`WORKTREE_GC_MAX_DAY`
(the script's own «authorized bulk rescue» path — a raised budget also skips sleep-leg pacing).
Locked worktrees with a stale `initializing` lock: `git worktree unlock` + `git worktree remove`,
never `rm -rf`; append the manual-finish row to the gc ledger. The breaker re-freezing itself
after the raised budget is consumed is its own protection — leave it; remaining SAFE-DELETE
branches (no disk weight) wait for the window.

## Phase 4 — Execute approved, verify

Exactly what was approved, nothing more. The two commands that differ most by OS:

```bash
find "$TMPDIR" -mindepth 1 -maxdepth 1 -mtime +0 ! -name opencode ! -name claude -exec rm -rf {} +
```

```bash
Get-ChildItem $env:TEMP | Where-Object Name -ne opencode | Where-Object LastWriteTime -lt (Get-Date).AddDays(-1) | Remove-Item -Recurse -Force -ErrorAction SilentlyContinue
```

Cache dirs: `rm -rf <dir>` (macOS) · `Remove-Item -Recurse -Force <dir>` (Windows). Then re-run
`ssd_measure.py` and report **baseline → now** per target — always the `df` delta, never the
`du` sum of what was deleted (APFS clones and open handles make the sum overstate it by 2x). On macOS `df` can trail the
deletions by a couple of GB until APFS purges its local snapshots (the census counts them).

**A file held by a live process:**
- macOS — quit the app that holds it (`lsof <file>` names it), then delete.
- Windows — the RunOnce trick: write `<home>\<name>_delete.cmd` with one
  `del /f /q "<path>"` per file and a last line
  `reg delete "HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce" /v <Name> /f >nul 2>&1`;
  arm it with `New-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\RunOnce" -Name "<Name>" -Value '<path to cmd>' -PropertyType String -Force`
  and read the key back. It fires at the next sign-in. (`schtasks /sc onlogon` is usually
  ACCESS DENIED from a non-elevated shell — go straight to RunOnce.)

Report: freed-now table per target + a pending-at-sign-in line (if armed) + final free space.

## opencode.db rescue — copy-exclude swap (both OSes)
## Proven runs
_Гасунс_
## References

Detail moved verbatim (read on demand — lives in the rendered skill copy's references/):

- `references/opencode-db-rescue.md` — opencode.db rescue — copy-exclude swap procedure (both OSes)
- `references/proven-runs.md` — Proven runs — measured before/after numbers
