---
name: m5-onboard
description: End-to-end onboarding for a freshly-plugged-in M5Stack ESP32 device (Cardputer, Cardputer-Adv, Core, CoreS3, Stick) — detect on USB, flash UIFlow 2.0 firmware, and install the Claude Buddy MicroPython app bundle. Use whenever the user plugs in or wants to flash/provision/reset an M5Stack or ESP32 board, or says "m5-onboard go".
---

# M5Stack Onboarding

This skill automates the full cold-start workflow for an M5Stack ESP32 device: detect on USB, identify model, flash UIFlow 2.0, and push a MicroPython app bundle onto `/flash/` so the device boots into user software. The apps we ship (Claude Buddy, Snake, Hello) talk over BLE or USB. The workflow runs on macOS, Linux, and Windows; the skill was developed against an M5Stack Basic v2.6 (CH9102 bridge, ESP32-D0WDQ6-V3, 16 MB flash) and generalized to cover the rest of the Core family, with the Cardputer-Adv (ESP32-S3, native USB) as the current default target.

## Where the scripts live

This skill ships as part of the `cwc-makers` plugin for reference, but the executable scripts and the `buddy/` app bundle live in a local clone of https://github.com/moremas/build-with-claude (the `/maker-setup` command creates this clone). Run every `scripts/*.py` invocation below from inside that clone's `onboard/` directory so `--apps buddy` resolves to the sibling `buddy/device/` payload.


## When to use

Use this when a user plugs in an M5Stack device and wants it provisioned. The decision tree:

- **Fresh/unknown device** → run `onboard.py --apps buddy` end-to-end (detect → identify → flash → install apps). This is the default path.
- **Already-flashed device, user just wants apps installed/refreshed** → run `install_apps.py --src buddy` (or any `--src <path>` to a directory of `.py` files).
- **Flashed device, something feels broken** → run `smoke_test.py` (I2C + LCD + speaker + button check).
- **User wants to know what's on the bus / what the device can do** → `smoke_test.py`.

If multiple devices are plugged in, ask which port to target — don't guess. If the user is provisioning a device they previously worked with (e.g. "same thing as last time" or "another Buddy"), default to `--apps buddy` unless they say otherwise.

### Which variant to assume

The rig this skill lives on provisions **Cardputer-Adv** boards overwhelmingly, so `onboard.py` now defaults to `--variant cardputer-adv`. In practice that means:

- If the user says nothing about the model, go with the default. They're almost certainly holding a Cardputer-Adv.
- If the user says "Cardputer" (no "Adv"), ask — the two models share a form factor but take different firmware images, and flashing the wrong one boot-loops the device.
- If the user names any other board ("Core2", "CoreS3", "Basic", "Fire"), pass the matching `--variant` explicitly — the default won't apply.
- The chip is ESP32-S3 either way, and `detect.py` won't be able to tell Cardputer from Cardputer-Adv before UIFlow is flashed (same native USB-JTAG VID, no pre-flash I2C probe). So this is a user-intent question, not a hardware-fingerprint one.

## The workflow

The main orchestrator is `scripts/onboard.py`. It drives the sub-scripts in order and handles the handoffs between them (waiting for reboots, capturing MAC, reporting progress). Prefer calling it directly over stitching the sub-scripts yourself unless the user asks for a partial run.

The default provisioning command (fresh Cardputer-Adv, install the buddy bundle):

```
python3 scripts/onboard.py --apps buddy
```

**How to invoke this from Claude Code's Bash tool.** Do NOT call `onboard.py` as a foreground Bash command. The Bash tool captures output and does not stream it back to the assistant until the command exits — and this command runs 2–3 minutes. That silence looks identical to a hang, and the assistant will usually give up before the button-dance prompt ever reaches the user. Instead, always run with `run_in_background: true`, `tee` to a log file, and then use the Monitor tool (or periodic `tail` via Read) to surface stage banners, heartbeats, and prompts to the user in real time. `2>&1` is not the fix — all progress already writes to stderr, which a terminal shows fine. The fix is streaming semantics, not redirection. The pattern that works:

```
# Launch (background, tee log):
python3 scripts/onboard.py --apps buddy 2>&1 | tee /tmp/m5-onboard.log

# Monitor (surfaces key events without drowning in byte-progress spam):
tail -f /tmp/m5-onboard.log | grep -E --line-buffered \
  "^====|heartbeat|Heads up|Enter download mode|download mode!|rebooted into UIFlow|Manual reset|DONE|ERROR|Error|Traceback|FAIL|failed|No USB|not detected|Attempt [0-9]|Device already in download|Download mode port|Post-flash port|Waiting for device"
```

### Relaying physical steps to the user (REQUIRED)

The flash stage **cannot proceed without a manual button press** on native-USB boards — there is no software path. When the monitored log shows `Enter download mode` (or the script appears to wait at the FLASH stage), you MUST stop and tell the user to do the following on the **back of the Cardputer**, in your own words, before continuing:

1. Press and **hold** the **G0** button
2. While still holding G0, briefly press and release the **RST** button
3. Keep holding G0 for about one more second, then release it
4. The screen should go fully dark — that means download mode is active

If the device reboots into UIFlow instead of going dark, tell the user G0 was released too early and to try again holding it longer. Do not move on, retry the script, or attempt a software workaround until the user confirms the screen is dark — the flash will not start otherwise. The same applies to any later `Manual reset` prompt: relay the physical step and wait for the user.

Users running `onboard.py` directly in their own terminal (not via Claude Code) will see all output live — no changes needed there.

If `--port` is omitted, `detect.py` picks the most likely candidate across all three OSes: native-USB ESP32-S3 (`/dev/cu.usbmodem*` on macOS, `/dev/ttyACM*` on Linux, `COMx` on Windows), or a CH9102/CP210x UART bridge on older boards. Bluetooth-serial ports are filtered out. If multiple candidates are present, it asks.

The known apps name `buddy` resolves to the `buddy/device/` directory in this repo (custom launcher + Hello + Claude Buddy BLE client + Snake). Any other `--apps` value is treated as a filesystem path.

To skip re-flashing and just push (or refresh) the apps onto an already-provisioned device:

```
python3 scripts/install_apps.py --port <PORT> --src buddy
```

Where `<PORT>` is whatever `detect.py` printed on the last full run — for example `/dev/cu.usbmodem1101`, `/dev/ttyACM0`, or `COM3`.

### Stages

1. **Detect** (`detect.py`) — enumerate serial ports, filter to USB-UART bridges (CH9102 vendor `0x1A86`, Silabs CP210x `0x10C4`, FTDI `0x0403`) or the ESP32-S3 native USB-JTAG interface (`0x303A`). Probe with esptool to confirm the chip. Port names differ per OS (`/dev/cu.usbmodem*` on macOS, `/dev/ttyACM*`/`ttyUSB*` on Linux, `COMx` on Windows) but pyserial abstracts that.
2. **Identify** (`detect.py`) — alongside port discovery, `detect.py` reads the factory-test partition signature and/or scans I2C once UIFlow is on, and cross-references `references/hardware_signatures.md` to suggest the right firmware variant (Basic-16MB, Core2, CoreS3, Cardputer-Adv, etc.). User-facing variant choice happens via `onboard.py --variant`; there is no separate `detect.py --identify` flag.
3. **Fetch firmware** (`fetch_firmware.py`) — query the M5Burner manifest API and download the appropriate UIFlow 2.0 binary into the system temp dir. Cached between runs — safe to clear the cache anytime, it just re-downloads.
4. **Flash** (`flash.py`) — `esptool write_flash 0x0 <image>` at **460800 baud** for UART bridges, `--no-stub` at 115200 baud for native-USB S3 devices. 921600 fails intermittently on the CH9102 bridge — do not increase it. Native-USB flash can intermittently throw `Lost connection, retrying` mid-erase; esptool recovers. The post-flash `watchdog-reset` teardown step can fail even when the flash itself succeeded — `flash.py` parses esptool's stdout, treats that specific failure pattern as non-fatal when `Hash of data verified` appeared, and `onboard.py` falls back to `flash.native_reset()` and then manual-RESET coaching if needed.
5. **Install apps** (optional, `install_apps.py`) — paste-mode REPL upload of every `.py` from a source directory into `/flash/`, then reboot via `repl_reset` (DTR/RTS is a no-op on native USB — don't reach for it). Source layout: root `*.py` → `/flash/`, `apps/*.py` → `/flash/apps/` (UIFlow's stock launcher scans that). When the bundle ships a root `main.py`, `install_apps.py` also sets NVS `boot_option=2` so UIFlow's own launcher doesn't run and our `main.py` takes over the boot flow — critical for BLE-using apps on ESP32-S3 (see gotchas below).
6. **Smoke test** (optional, `smoke_test.py`) — I2C scan, LCD test pattern, speaker beep, button read.

## Files

- `scripts/onboard.py` — main orchestrator
- `scripts/detect.py` — port discovery + chip ID
- `scripts/fetch_firmware.py` — M5Burner API + download
- `scripts/flash.py` — esptool wrapper
- `scripts/install_apps.py` — push a directory of `.py` files into `/flash/` via paste-mode REPL; backs up `boot.py` as `boot_uiflow.py` before overwriting; also writes the `boot_option` NVS key when the bundle ships a root `main.py`
- `scripts/smoke_test.py` — I2C + LCD + speaker + buttons
- `scripts/mpy_repl.py` — shared serial/REPL helpers (paste mode, hard reset, boot-log capture)
- `references/hardware_signatures.md` — chip + I2C fingerprints → model → firmware
- `references/uiflow2_nvs.md` — NVS key reference with types and failure modes

## References

Content split (H5588) — sections below were moved verbatim, read on demand:

- [references/critical-gotchas.md](references/critical-gotchas.md) — Critical gotchas baked into the scripts — read before deviating from onboard.py/install_apps.py
- [references/after-provisioning.md](references/after-provisioning.md) — What the user sees on the device after provisioning (power, boot, launcher, BLE pairing, UIFlow revert)
- [references/dependencies.md](references/dependencies.md) — pyserial vendoring, esptool pip install, per-OS Python bootstrap, USB drivers
- [references/platform-notes.md](references/platform-notes.md) — Per-OS port naming, Linux dialout permissions, Windows PATH/Store gotchas, bundle path resolution, firmware cache
