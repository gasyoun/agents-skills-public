## Change log

Newest first. This section exists so the metadata line at the top can stay two dates long.

| Date | Change |
|---|---|
| 24-09-2026 | H5237: passive media-key diagnostic probe (logs systemDefined payloads to both hit logs, never dictates, never swallows; real keys = `subtype 8`, keycode in `data1>>16`) + **unconditional** wake hotkey re-assertion (object `:delete()` → rebind both → log; honest scope = registration, delivery proven only by the trace) + probe self-heal in `healthCheck` — closes the 21-09 unprobeable-theft + silent-delivery gaps; lane selftest 58 PASS, DeepSeek-verified |
| 22-09-2026 | H5246: owner-tagged hung-recorder watchdog (stale-watchdog race = the random banner storm), quiet mode (error-only alerts), `dictate_selftest.sh` 42-check lane selftest (incl. symlinked-`/tmp` armed-file sweep, no-litter check) with a red-before-green mutant |
| 22-09-2026 | H5249: script-side quiet-hours gate — `dictate.sh` refuses a NEW take 22:00–08:00 silently (Carbon F5 + ⌥D were ungated); stop still works; selftest 49 checks |
| 21-09-2026 | H5236: recurring-death audit — watchdog v3 health = IPC `print(1)` **and** ≥2 registered hotkeys (catches the IPC-alive/hotkey-dead zombie class), `f5hits` dual-write to `/tmp` + `~/Library/Logs` + config-load discriminator, breaker vendored to `~/bin` as sh+py+pyfloor (launchd TCC cannot read `~/Documents`), FROZEN/RECOVERED-failed scream outside the guarded process |
| 17-09-2026 | F5 3-day death #2: OS-level key theft (Dictation On / fnState media mode) + full hardening — hourly theft probes, paste commit traces, repetition-based hallucination refusal, dynamic timeout, load downshift to small, weekly model bench |
| 15-09-2026 | F5 die-every-day root cause: Lua GC trap in the debounce latch — incl. the heap-pressure repro precondition and the trace-at-commit-point rule |

