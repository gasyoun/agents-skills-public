
| Date | Box | Result | What it taught |
|---|---|---|---|
| 08-09-2026 | Windows (MSI) | C: 0.7 → 25 GB free + db 107 → 12 GB | robocopy measuring; prune + VACUUM + swap; RunOnce |
| 10-09-2026 | Mac | 99 → 107 GB free | `uv cache clean` hangs; `pip cache purge` misses the Mac dir |
| 11-09-2026 | Windows | C: 0 → 4.2 GB free + db 115 → 4.13 GB | copy-exclude beats prune; rehearse the kit first; census every drive |
| 12-09-2026 | Windows (MSI) | new sessions dead after a swap | keep `event` DDL; write-path probe with auto-restore (H4612) |
| 15-09-2026 | Mac | db 82 → 2.6 GB, 26 → 97 GB free | holder check = `lsof`, not `pgrep`; the surviving session is the check; caches regrow |
| 15-09-2026 | Mac + Windows | kit moved into claude-config, one Python driver for both OSes | live-copy orphan parts → one-snapshot compact; bench 12/12 on both boxes (H4898) |
| 15-09-2026 | Mac + Windows | independent review (Sonnet 5) found 4 rare failure paths | half-failed move could delete the original `-wal`; Ctrl-C skipped restore; Windows rename-back failure misreported; dry-run rw fallback — all fixed, bench 16/16 on both boxes |
| 24-09-2026 | Mac | 2.1 → 17 GB free (Tier 1 + 1 ollama model + 7 worktrees); db 45.9 GB, dry-run projects 41.6 GB | `du` sum overstated the `df` gain 2x (uv APFS clones); census worktree row 65.7 GB ≠ 2.2 GB removable; db regrew ~5 GB/day; dry-run now deletes its copy; 4 new census rows; real swap aborted on an OpenCode relaunch mid-copy (parity 87029≠87027) — abort now deletes the 4.7 GB partial copy (Opus 5.5) |
| 17-09-2026 | Windows | C: 0 → 35 GB free (two concurrent passes; 15 worktrees via hand-run Phase 3) | full disk masqueraded as «download failed»; stray `oc_*.db` pair (16.7 GB) was the #1 offender; MG ruling: db swap not worth it at 12.4 GB |

