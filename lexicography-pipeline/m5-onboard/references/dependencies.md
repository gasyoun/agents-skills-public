## Dependencies

- `pyserial` — vendored at `onboard/scripts/vendor/serial/` (pinned 3.5, BSD-3-Clause).
- `esptool` — pip dependency, declared in `requirements.txt`. Importable check happens via `importlib.util.find_spec("esptool")`; binary backstop search covers `~/Library/Python/*/bin/` on macOS, `~/.local/bin/` on Linux, `%APPDATA%\Python\Python3XX\Scripts\` on Windows.

`onboard.py` runs a preflight check at startup: if `esptool` (or, in the rare prune-vendor case, `pyserial`) is missing, it lists what's needed and asks the user whether to install now. On `Y` (or Enter) it runs `python -m pip install --user <missing>` in the current interpreter, then verifies. Inside a venv the `--user` flag is dropped so the install lands in the venv's site-packages. Non-interactive callers (piped stdin) get a manual-install hint instead of a prompt.

Python itself has to exist before this skill can do anything — you can't bootstrap an interpreter from inside one. `git` is **not** required — the `/maker-setup` command falls back to downloading the GitHub tarball with `curl`+`tar` (both pre-installed on macOS, Linux, and Windows 10+) when `git --version` fails. Claude's responsible for detecting Python and installing it if missing *before* running any `scripts/*.py` invocation. Detection is just running `python3 --version` / `python --version` — if it fails, Claude fetches Python with the host's native package manager before anything else.

**Per-OS Python bootstrap (Claude's responsibility if missing):**

- **Windows** — `winget install -e --id Python.Python.3.13 --silent --accept-source-agreements --accept-package-agreements`. Takes ~30 seconds, no UI, gets PATH right. If the current shell can't see `python` afterwards, tell the user to close and reopen the terminal (Windows updates PATH only on new shells).
- **macOS** — Python 3 is usually pre-installed as `/usr/bin/python3` on any current macOS (shipped by Apple). If for some reason it isn't, `brew install python@3.13` via Homebrew is the go-to; if Homebrew itself is missing, offer to install it via `/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"` (but only if the user confirms — Homebrew is a larger commitment than winget).
- **Linux** — use the distro package manager. Debian/Ubuntu: `sudo apt-get update && sudo apt-get install -y python3 python3-pip`. Fedora: `sudo dnf install -y python3 python3-pip`. Arch: `sudo pacman -S --noconfirm python python-pip`. You may need to sudo and should surface the password prompt to the user if needed.

**pyserial — bundled with the skill:**

A pinned `pyserial 3.5` ships under `scripts/vendor/` (BSD-3-Clause, Apache-compatible). Every script that imports `serial` calls `vendor_path.ensure_on_syspath()` before the first third-party import, which prepends `scripts/vendor/` to `sys.path`, so the vendored copy resolves regardless of whatever the user has system-wide. Net effect: port enumeration and REPL I/O work on a fresh clone with zero pip step. ~500 KB, pure-Python, same tree on macOS / Linux / Windows.

**esptool — pip dependency, auto-installed on first run:**

`esptool` is GPLv2+ and is intentionally **not** vendored — keeping the repository cleanly Apache-2.0 means the GPL bits live in the user's pip-managed environment, not in the tree. The skill's preflight checks for an importable `esptool` and, if missing, prompts to install it (`python -m pip install --user esptool` — `--user` dropped inside a venv so it lands in site-packages). For subprocess calls we use `[sys.executable, "-m", "esptool", ...]`; the subprocess inherits user-site so the pip-installed module imports cleanly. `requirements.txt` declares this for explicit setup; the prompt path is the default for first-time attendees who haven't run pip yet.

Non-interactive callers (piped stdin, CI) skip the prompt and get a `python -m pip install --user esptool` hint instead.

**Fallback if someone prunes `scripts/vendor/`:**

The same preflight path also re-installs pyserial via pip if the vendor copy is gone. This handles the case where someone downloaded a source-only zip that excluded vendor, or manually trimmed the repo to save space.

**USB driver — Windows-specific, only for older boards:**

The CH9102 USB-UART driver is still a manual install on Windows — WCH doesn't publish a winget manifest. Only needed for UART-bridge boards (Basic, Fire, Core2, StickC). Native-USB ESP32-S3 boards (Cardputer, Cardputer-Adv, CoreS3) enumerate as composite USB-CDC devices using Windows' in-box drivers and need no extra install.

