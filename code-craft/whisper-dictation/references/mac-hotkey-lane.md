## Mac hotkey lane (always-on dictation — M3 Mac)

Always-on push-to-talk, NOT the python script above. Press **F5** or **Option+D** →
record → press again → transcribe (RU/EN auto, `ggml-large-v3-turbo`) → paste at cursor.

| Part | Location |
|---|---|
| Toggle script | `~/bin/dictate.sh` (whisper-cli + ffmpeg + Hammerspoon IPC paste) |
| Hotkeys + paste | `~/.hammerspoon/init.lua` (`hs.hotkey.bind` F5 + ⌥D — Carbon-level, works WITHOUT Input Monitoring permission; eventtap approach is dead on this box) |
| Watchdog (v3) | `~/bin/hammerspoon-watchdog.sh` — launchd, every 180 s; healthy = IPC `print(1)` **and** ≥2 registered hotkeys, heartbeat `/tmp/hs-watchdog.heartbeat`, breaker-vendored killall, screams outside the guarded process |
| Model | `~/.whisper/models/ggml-large-v3-turbo.bin` (~1.6 GB) |
| Autostart | Hammerspoon is a macOS login item |

**If anything breaks or was swept away** — one command restores everything
(configs rewritten from embedded copies, model re-downloaded only if missing):

```text
~/bin/whisper-dictate-restore.sh
```

Gotchas (06-09-2026): Option+Space is reserved (was colliding with user muscle memory);
language switch = Ctrl+Space; Hammerspoon lacks Input Monitoring permission — never
reintroduce `hs.eventtap` for the F5 path, use `hs.hotkey.bind`.

Gotchas (14-09-2026, F5 "dead" root cause): **launchd PATH (/usr/bin:/bin:/usr/sbin:/sbin)
has no `/opt/homebrew/bin`** — a launchd-run script calling bare `hs` fails with
«command not found». The zombie IPC-watchdog (`~/bin/hammerspoon-watchdog.sh`, added
12-09 after a real HS crash) misread that broken probe as «HS is a zombie» and
killall+relaunched Hammerspoon every 3 min for 2 days (**870 kills**, 12-09 16:10 →
14-09 13:57) — F5 dictation died mid-keystroke, alerts/paste vanished, recorders got
orphaned. Two standing rules for EVERY launchd-run guard script: (1) set
`PATH="/opt/homebrew/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin"` explicitly at
the top — launchd never inherits your shell PATH; (2) **fail-safe direction: a probe
you cannot run is UNVERIFIABLE — log and skip, never kill** (only a real wrong/hung
IPC answer may condemn the instance). Fix lives on the Mac box:
`~/bin/hammerspoon-watchdog.sh` (PATH + `command -v hs` skip-guard + bounded recovery
check) and an embedded copy in `~/bin/whisper-dictate-restore.sh` §4b, so a restore
reinstalls the fixed watchdog. Recovery probe: tail
`~/Library/Logs/hammerspoon-watchdog.log` — a healthy cycle is SILENT; any `ACTION
killall` line every ~180 s = the loop is back. Same-day v2 (protection that cannot
fail silently): the watchdog has a **circuit breaker** — max 2 killall/hour,
4/24h, then it freezes itself and raises a loud alarm (macOS notification + `say`);
it stamps a heartbeat (`/tmp/hs-watchdog.heartbeat`) every healthy cycle, and
`init.lua` healthCheck (now hourly) alerts when the heartbeat goes stale >2h —
guard-the-guard, so a broken guard announces itself instead of quietly eating the
lane. A 🎤 icon in the menu bar = dictation alive (🔴 recording, ⏳ busy; icon
gone = Hammerspoon down). Negative selftest on record: fake broken `hs` →
`ACTION 1/2 → ACTION 2/2 → FROZEN` with the real HS untouched (test hooks:
`HS_BIN`, `WD_STATE_DIR`, `WD_TEST_MODE=1`).

Gotchas (15-09-2026, F5 die-every-day root cause — **a Lua GC trap, not a hardware
or permission problem**): `init.lua`'s debounce used
`hs.timer.doAfter(1.5, function() busy = false end)` and never stored the returned
timer object. **Hammerspoon's `hs.timer` releases the callback unless the timer is
referenced** — so any Lua GC pass landing inside the 1.5 s window collected the
reset timer, `busy` stayed `true` for the rest of the Hammerspoon session, and every
later F5/M5 press returned at the guard. The failure signature is uniquely confusing
because the hit trace stayed GREEN: `/tmp/f5hits.log` is written at the TOP of
`dictate()` (by design, to attribute key-delivery), so the log proved the key reached
Hammerspoon while `dictate.sh` never dispatched — no error, no crash report, no alert
(silent `return`), mic/permission/whisper all healthy. ⚠️ **Reproducing it needs the
right precondition — an idle repro proves nothing.** Measured on this box 15-09-2026,
three conditions, same Lua state, run serially: unretained `doAfter` with the heap left
**idle** fired **100/100** (the defect does NOT show); the same 100 unretained timers
under **heap pressure** (400k-string churn) fired **0/100**; and with an explicit
`for k=1,5 do collectgarbage("collect") end` fired **0/100**. The trigger is **heap
pressure, not elapsed time** — so the honest statement is «collectable when a GC pass
happens to land inside the window», which is why it read as *every day* rather than
*always*, and why a first repro attempt that only schedules unretained timers reports
a false negative. Retained control fired **100/100** under the identical forced-GC loop.
Attribution by intervention: `hs.reload()` (fresh `busy=false`) → next press dispatched
immediately, re-confirming latch-not-permissions. **Standing rules — ANY debounce or
latch, ANY language, and ANY Hammerspoon Lua timer:**
(1) **store the timer object** (`local t = hs.timer.doAfter(...)`, keep it in a
module-level/`_G` table) or don't rely on the callback at all; (2) **prefer a pure
timestamp compare over a timer-based latch** when debouncing — arithmetic cannot be
garbage-collected; (3) **a trace must be written where the side effect is COMMITTED,
never at handler entry** — this entry is the proof: the hit trace sat at the top of
`dictate()` and stayed green for a whole day over a dead lane, because entry proves the
key ARRIVED, not that anything RAN. A probe at entry measures reachability; only a probe
at the commit point measures function; (4) **a guard that returns silently is
unobservable** — log the rejection (`declined (reason)` beside the entry line), or a
correct debounce and a hang are byte-identical in every log on the box. Fixed on the Mac box as a timestamp compare
(`busy = hs.timer.secondsSinceEpoch()`, `BUSY_WINDOW = 1.5`) with the debounce
rejection now LOGGED (`debounced (within 1.5s of previous press)` in
`/tmp/f5hits.log`) so a future silent block is attributable in one glance instead of
a debugging session; the `healthCheck` `doAfter`/`doEvery` pair is retained in
`_G.__dictateHealthTimers` for the same reason. Same trap applies to
`whisper-dictate-restore.sh`'s embedded copies — both live copies synced by
construction (embedded block rewritten from the live file). Diagnostic shortcut for
the next report of «F5 не работает»: check whether a hotkey-fires line has a matching
dispatch within the same second (`stat -f '%Sm' /tmp/dictate.log`) — fires-without-
dispatch = the guard/latch class, dispatched-but-no-paste = the paste/IPC class.
Full finding (with the negative-control table): [Uprava FINDINGS §776](https://github.com/gasyoun/Uprava/blob/main/FINDINGS.md).

Gotchas (17-09-2026, F5 3-day death #2 — **the OS layer silently re-keys the
physical key; a healthy bind receives nothing**): the fn-key row had been reset to
media mode, so physical F5 emitted the macOS **Dictation media keycode**, never an
F5 keycode — Hammerspoon's healthy Carbon bind saw nothing, and the entry trace
went SILENT. Attribution rule (completes the 15-09 pair): green `/tmp/f5hits.log`
= key arrived, problem downstream; **silent log = OS-level grab**. Two GUI toggles
own this, both invisible to any config check: (1) System Settings → Keyboard →
**Dictation → Off** (when On, Apple dictation grabs F5 first and shows its own UI —
the 16-09 «accidental F5 triggers mic+chime» note was this, misdiagnosed, and led
to removing our bind); (2) Keyboard Shortcuts → Function Keys → **«Use F1, F2 as
standard function keys» → ВКЛ** (media mode = the F5 row emits media codes).
⚠️ `defaults read com.apple.keyboard.fnState` stays UNSET even after flipping it —
per-keyboard storage, useless as a global probe. Hardening landed 17-09 (MG
rulings «Restore F5» + «чтобы ни при каких обстоятельствах никогда не лопалась» +
«лучше хуже распознать, чем вообще потерять»): `healthCheck` probes the theft
class hourly (hotkey count < 2; `Dictation Enabled` ≠ 0 → loud alert with the exact
treatment); `dictatePaste` logs the paste COMMIT point (`pasted N chars` /
`empty transcript` / `no /tmp/dictate.text`) so dead-key, dead-paste and
nothing-said are three distinct signatures; hallucination refusal is
**repetition-based, not length-based** (unique 40-char chunks < 35% = whisper loop
wall → clipboard + file only; a genuine 30-min dictation is ~25K unique chars and
types in full — selftest 44K wall=refuse, 36K real=type); whisper timeout is
**dynamic** (2× audio duration + 120 s — a starved turbo crawled at ~7% CPU for
6+ min on 7 s of audio and held the lane); **load downshift**: load ≥ 0.5/core →
switch to `ggml-small` BEFORE transcribing with a visible «машина перегружена»
note; every transcript is saved to `/tmp/dictate-last.txt` before typing
(losslessness); `~/.hammerspoon` is a git repo (the 16-09-style silent edit is
`git diff`-visible); a missing `}` in a notify table killed the whole config
INCLUDING `hs.ipc` (Lua compiles the chunk before running) → **validate with
`luac -p init.lua` BEFORE reload**, dead IPC port + alive process = broken config,
not dead app. Weekly model bench: `~/bin/whisper_bench.py` (4 models × Milena-TTS
RU corpus, RTF + CER, launchd Sunday 11:00, reports `~/whisper-bench/reports/`) —
first run 17-09: turbo CER 0–0.3% at RTF 0.15–0.44, keep as default; `small` is
the load fallback. Full story: [Uprava memory note
17-09](https://github.com/gasyoun/Uprava/blob/main/.claude/projects/Uprava/memory/2026-09-17-f5-dictation-system-key-theft-fnstate-media-mode.md).

Gotchas (21-09-2026, H5236 — **the recurring-death audit: one symptom lived on ~5
alternating layers, and telemetry died at every reboot**): F5 was dead on 20 days out
of 21, and each day was re-diagnosed from zero because `/tmp` is wiped by a reboot
(three wipes: Sep 6/15/21). (1) **Watchdog v3 health is TWO probes, not one** — IPC
`print(1)` **AND** `#hs.hotkey.getHotkeys() >= 2`. On 21-09 the IPC answered `print(1)`
all day while the hotkey layer was dead (zero `/tmp/f5hits.log` lines despite presses),
so a print(1)-only probe blessed a zombie from boot 04:48; a healthy cycle now renews
`/tmp/hs-watchdog.heartbeat` only when both layers answer, recovery asserts both
(3 attempts × 5 s), and when the `hs` CLI SIGSEGV'd during a watchdog ping that day
(`DiagnosticReports/hs-2026-09-21-172410.ips`, `EXC_BAD_ACCESS`) the watchdog restarted
it → `RECOVERED ok`. Negative selftest on record (fake `hs` answering IPC with
`hotkeys=1`, `WD_TEST_MODE=1`): `DEAD zombie-hotkey: IPC answers but hotkey layer dead
(<2 registered)` → `ACTION killall + relaunch (breaker allowed)` → `RECOVERED failed —
needs human` → scream. (2) **Persistent telemetry**: `traceHit` dual-writes
`/tmp/f5hits.log` **and** `~/Library/Logs/f5hits.log` (reboot-surviving) plus a
config-load discriminator line (`config loaded, hotkeys=2`), so every layer's failure
is attributable in one glance instead of a fresh investigation; `~/.hammerspoon` commit
`5f80516`. (3) **The breaker must live next to the guard**: launchd's TCC cannot read
`~/Documents`, so on 17-09 the watchdog's call into the Uprava `sentinel_breaker` died
with `Operation not permitted` + `sb_check: command not found` and the guard stood down
**FROZEN** on the exact day the lane was dead. The breaker is now vendored into `~/bin`
as a **triple** — `sentinel_breaker.sh` + `sentinel_breaker.py` + `pyfloor.py` — and
auto-refreshed from the Uprava canonical when readable. Vendor the whole closure: a
sh-only vendor froze the breaker in selftest #1 and a py-without-pyfloor vendor froze it
in #2 (the `.py` resolves its companions next to itself); only the full triple lets the
breaker allow the kill. (4) **Scream outside the guarded process**: `FROZEN` /
breaker-missing / `RECOVERED failed` → macOS notification + `say -v Milena`, once per
incident (the dedupe marker is cleared by the next healthy cycle) — a guard that
announces itself through the thing it guards goes silent exactly when it matters.
(5) The restore kit `~/bin/whisper-dictate-restore.sh` was re-spliced byte-identical
(`init.lua` 11849 B, watchdog 9901 B — UTF-8 bytes; the earlier 11437/9576 figures were
code-point counts). The GC-hotkey hypothesis was tested live and **REJECTED**
(`hs -c 'collectgarbage("collect") collectgarbage("collect") return #(hs.hotkey.getHotkeys())'`
→ 2), so the 21-09 daytime class is key delivery (boot race / media mode), not
unanchored hotkey objects. Receipts:
[reports/F5_DICTATION_HARDENING3_EVIDENCE_21-09-2026.md](https://github.com/gasyoun/Uprava/blob/main/reports/F5_DICTATION_HARDENING3_EVIDENCE_21-09-2026.md);
audit [H5236](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H5236-OxAlpha_Uprava_f5-dictation-hardening3-telemetry-watchdog-v3_21.09.26.md).

Gotchas (22-09-2026, H5246 — **the «random» banner storm was a watchdog killing the
NEXT take**): the hung-recorder watchdog added that morning (a 12 s subshell that
kills a recorder whose wav stays under 40000 B) matched its target by PATTERN
(`pkill -9 -f 'ffmpeg -f avfoundation.*dictate.wav'`) and deleted the pidfile
unconditionally. A take stopped inside its 12 s window left the subshell alive; when
the next press started a new recorder, the stale subshell saw a fresh, still-small wav
and killed the new take mid-recording, then showed «Запись зависла» (36 recorder starts,
14 presses/min at 10:26 in `~/Library/Logs/f5hits.log`). Fix: the watchdog is
**owner-tagged** — it captures the recorder PID (`RECPID=$!`) before the pidfile is
written, kills only that PID (`kill -9 $RECPID`), clears the pidfile only if it still
holds that PID, and exits early when its recorder is gone, the PID was recycled (cmdline
no longer matches), or its per-take ARMED file `/tmp/dictate-wdog-armed.<pid>` is gone.
The start path creates that file; the stop path removes it BEFORE killing the recorder (that
removal is the cancel); the watchdog removes it on every exit. An earlier version used a
cancel flag that the stop path touched, which left litter whenever the stop came after the
watchdog had already exited (verifier round 2). Only a watchdog killed from outside can
orphan an armed file; those are swept at start after 10 min with **`find -H`**, because
`/tmp` on macOS is a SYMLINK to `/private/tmp`. Plain `find /tmp -maxdepth 1` does not
descend a symlinked start point. The first version passed its selftest (plain temp dir) and
still matched nothing live; the independent DeepSeek verifier caught that from flags left in
`/tmp`, and the selftest's state dir is now a symlink too. The hang window is 6 ticks × 2 s,
so ≥12 s; stat and ps per tick stretch it under load (measured up to ~2×). **Quiet mode**
(MG «тихий»): `alert()` is error-only — the per-cycle «🔴 Идёт запись», «⏳ Распознаю»,
«✅ Вставлено» banners are gone; the menubar 🎤/🔴/⏳ flips carry the state, and error
alerts stay (Запись зависла · Ничего не распознано · Запись не удалась · Ещё
расшифровываю · таймаут-частичный текст). **Lane selftest:** `~/bin/dictate_selftest.sh`
(embedded in the restore kit) drives the real `dictate.sh` in `DICTATE_SELFTEST_DIR` mode:
private temp dir, fake recorder, fake whisper, hs and pbcopy stubbed to a log, never a sound.
It has 42 checks: kit byte-identity, `luac -p`, live interceptor state, the TLOCK start gate,
the readability gate, kill -9 escalation, the watchdog positive, the stale-watchdog race
(normal stop and orphaned), the armed-file orphan sweep and no-litter check, quiet mode and quiet hours. A live probe that cannot run (Hammerspoon IPC wedged under load) is reported as UNKNOWN, not FAIL. `--mic` adds an opt-in
2 s mic probe (it lights the orange mic dot). **Red-before-green on record:** a mutant with
the old pattern-kill watchdog fails the race sections (10 FAIL: take B killed,
«Запись зависла» fired). The fix passes 42/42 (a no-`-H` mutant fails both [5d] checks). The watchdog writes one line per take to `/tmp/whisper-dictate.log`: `watchdog <pid>: stood down` / `wav healthy` / `hung … — killed`. The selftest waits on those lines, never on fixed sleeps, after a fixed 4.5 s wait flaked [5a] for the verifier on this overloaded box. Evidence: `~/Library/Logs/dictate_selftest.log`.
Quiet hours, both layers (22-09-2026, H5249, MG «да»): the 22:00–08:00 swallow first
lived only in the media-key interceptor, so in standard-F-key mode the Carbon F5 bind (and
⌥D) still dictated at night. `dictate.sh` now gates its START branch too: a new take is
refused silently between 22:00 and 08:00, with one `quiet hours (Nh): start refused silently`
line in `/tmp/whisper-dictate.log` and no banner or menubar flip. STOP is not gated, so a
take begun at 21:59 still ends and pastes. The hour is parsed as `10#` (08 and 09 are not
octal). `DICTATE_HOUR` overrides the hour only in `DICTATE_SELFTEST_DIR` mode; the selftest
pins it to 12 and its [7] section adds 7 night checks (49 total). A mutant without the gate
fails 5 of them.
Since 22-09 16:45 the media-key *interceptor* is gone (`mediaKeyEventTap:start()` removed after left-click mis-fires); media-mode F5 reaches the Carbon bind through a `hidutil` remap of the mic key (0xC000000CF → F5), so the Lua swallow is dormant and the script-side gate is the ONLY night gate. Since 24-09 (H5237) a **passive probe** on the same systemDefined stream is back — see the paragraph below.

Gotchas (24-09-2026, H5237 — the two gaps left open 21-09: **media-mode theft was unprobeable, and the hotkeys could silently fail delivery**): (a) The 22-09 interceptor was disabled because it matched `subtype=7, data1=1, data2=0` — which is a **left-mouse-button release** (NX_SUBTYPE_AUX_MOUSE_BUTTONS), not the dictation key — and it **swallowed** the event and called `dictate()`, so every click toggled a recording. The price of that retreat was that **no diagnostic survived**: when the physical F5 later went silent in media mode, nothing was written anywhere and the theft class was unattributable. H5237 rebinds it as a **passive probe**: an `hs.eventtap` on `systemDefined` that **never** calls `dictate()` and **never** swallows (always `return false`), writing only the raw payload (`media-probe subtype=… data1=… data2=… keycode=… state=…`) to both hit logs. Real consumer/media keys arrive as `subtype=8` (NX_SUBTYPE_AUX_CONTROL_BUTTONS) with the keycode in the high 16 bits of `data1`; mouse noise (`subtype=7` with no keycode) is ignored, so a click can never pollute the log. A silent log during a known media-mode press is itself evidence — the event was consumed below us (Apple Dictation) or never emitted. Two standing lessons: a **probe must never be the thing that acts** (the old tap conflated observation with an action), and a **disabled detector is a deleted detector** — when a mechanism is turned off, leave a passive sensor behind or the next failure is unattributable. (b) The wake handler (`hs.caffeinate.watcher`) already re-asserted the `hidutil` mic-key remap on wake/unlock; it now **unconditionally re-asserts hotkey registration** on every wake/unlock — delete-all via the hotkey object's `:delete()`, then re-bind both F5 + ⌥D, logging `hotkeys RE-ASSERTED on wake (count N -> M)` and alerting if the fresh count is still < 2. An independent verifier (DeepSeek, H4934 gate) flagged that a count-conditional rebind would miss the exact 21-09 shape (count=2, delivery dead), so the rebind is unconditional. ⚠️ **HONEST SCOPE:** this guarantees fresh REGISTRATION; it cannot prove DELIVERY — only the f5hits trace proves delivery, and the registration count is never treated as a stand-in. The probe is **self-healing**: `healthCheck` re-starts it and alerts if the eventtap is not enabled (a disabled probe = the theft class goes unprobeable again). ⚠️ This box's `hs.hotkey` exposes **no per-key module delete** (`hs.hotkey.deleteAll` errors with no live hotkeys) — clear via the hotkey object's `:delete()` (`for _, hk in ipairs(hs.hotkey.getHotkeys()) do hk:delete() end`, proven live: `count 1 -> 2` and `count 2 -> 2`). Fake-event selftest hook: `_G.__mediaKeyProbeTest({subtype=8,data1=65536,data2=0})` returns `true` and writes a log line; `{subtype=7,data1=1,data2=0}` returns `false` and writes nothing. Lane selftest `~/bin/dictate_selftest.sh` **58 PASS / 0 FAIL / 0 UNKNOWN**; `luac -p` before every reload; restore kit re-spliced byte-identical ([0] asserts it, sha256 in [the evidence report](https://github.com/gasyoun/Uprava/blob/main/reports/EVIDENCE_H5237_F5_MEDIA_PROBE_24-09-2026.md)).
