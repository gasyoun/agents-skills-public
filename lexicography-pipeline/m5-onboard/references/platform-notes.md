## Platform notes

The skill runs on macOS, Linux, and Windows. Non-obvious bits:

- **Port naming.** pyserial abstracts the lookup but what the user sees looks different per OS. Pass whichever form `detect.py` reports:
  - macOS: `/dev/cu.usbmodem1101` (native USB) or `/dev/cu.usbserial-XXXX` (CH9102)
  - Linux: `/dev/ttyACM0` (native USB) or `/dev/ttyUSB0` (UART bridge)
  - Windows: `COM3`, `COM4`, etc. (Device Manager → Ports if unsure)
- **Linux permissions — read this before blaming hardware.** On most distros, accessing `/dev/ttyUSB*` / `/dev/ttyACM*` without sudo requires group membership (`dialout` on Debian/Ubuntu/Arch, `uucp` on Fedora). Symptom: `detect.py` finds the port, but the flash step fails with `Permission denied` or `Could not open port`. Fix once, long-term:
  ```bash
  sudo usermod -aG dialout $USER
  # log out / log back in — group change only takes effect for new sessions
  ```
  `sudo python3 scripts/onboard.py ...` works as a one-off but adding the group membership is strictly better because pyserial's port-open in user mode succeeds cleanly from then on.
- **Windows PATH gotchas.** Python's `pip install --user esptool` lands the executable in `%APPDATA%\Python\Python3XX\Scripts\`. If that directory isn't on PATH, `pip` prints a warning and nothing else picks up the install. `detect.py` looks there directly as a backstop, so the skill still works even without PATH fixed. But if you're invoking esptool outside the skill (or hitting "esptool not found" errors from other tools), either:
  - Re-run the Python installer and tick "Add Python to PATH" (the install's default), OR
  - Add `%APPDATA%\Python\Python3XX\Scripts` to PATH via System Properties → Environment Variables, OR
  - Use `python -m esptool ...` which always works regardless of PATH.
- **Windows Store Python.** Newer Windows 11 machines may have Python pre-installed via Microsoft Store. It works but has quirky PATH behavior (lives under `%LOCALAPPDATA%\Packages\PythonSoftwareFoundation.Python.*\`). `detect.py` checks that location too. If you have the choice, the `winget install Python.Python.3.13` version is more predictable.
- **Bundle path resolution.** `install_apps.py`'s `--src buddy` shorthand resolves in this order:
  1. `$M5_BUDDY_DIR` if set — explicit override, always wins. Useful when you want to point at a fork or a customized bundle that isn't in this clone.
  2. The `buddy/device/` directory inside this repo, found via `os.path.realpath(__file__)` walking up from `install_apps.py`. Works for any clone location, including symlinked skill installs at `~/.claude/skills/m5-onboard/`.
  3. `~/Downloads/m5stack/buddy/device`.
  4. `~/Desktop/m5stack/buddy/device`.

  Most installs hit (2). Set `M5_BUDDY_DIR` only for the unusual case of pointing at a bundle outside this clone: `export M5_BUDDY_DIR=/path/to/buddy/device` (Unix) or `$env:M5_BUDDY_DIR="C:\path\to\buddy\device"` (PowerShell).
- **Firmware cache.** Downloaded firmware lands at `~/.cache/m5-onboard/` (or `$XDG_CACHE_HOME/m5-onboard/`), created at mode 0700 if missing. Cache files are MD5-verified at write time and re-verified on hit. Clearing the cache is safe; the next run re-downloads.
