---
name: "whisper-dictation"
description: Local Whisper (faster-whisper) voice dictation for long Claude Code briefs — record mic, transcribe, clipboard. Use when the user says "whisper", "dictate", "voice brief", "voice dictation", "record a prompt", or wants spoken instructions pasted into Claude.
---

# whisper-dictation

Use this skill when the user asks to run the migrated source command `whisper-dictation`.

## Command Template



# /whisper-dictation [--seconds N] [--model base|small] [--language ru|en]

_Created: 01-08-2026 · Last updated: 01-10-2026_

Script: [`scripts/whisper_dictation.py`](https://github.com/gasyoun/claude-config/blob/main/scripts/whisper_dictation.py).

## Default path (seconds — run this first)

```text
python ~/.claude/scripts\whisper_dictation.py        # Windows
python3 ~/.claude/scripts/whisper_dictation.py       # macOS
```

Optional:

```text
python ~/.claude/scripts\whisper_dictation.py --seconds 45 --model small --language ru
python ~/.claude/scripts\whisper_dictation.py --list-devices
python3 ~/.claude/scripts/whisper_dictation.py --seconds 45 --model small --language ru   # macOS
python3 ~/.claude/scripts/whisper_dictation.py --list-devices                              # macOS
```

Script prints the transcript and copies it to the clipboard when `pyperclip` works. Paste into Claude Code (or the agent pastes via user action).

## Setup (one-time)

```text
python -m pip install faster-whisper sounddevice soundfile numpy pyperclip
```

First transcription downloads the model (`base` ≈ 75 MB). CPU uses `int8`.

macOS (one-time): sounddevice speaks to CoreAudio through PortAudio, so install that
first; everything else is identical:

```sh
brew install portaudio
python3 -m pip install faster-whisper sounddevice soundfile numpy pyperclip
python3 ~/.claude/scripts/whisper_dictation.py --list-devices   # CoreAudio devices listed
```

`pyperclip` on macOS shells out to `pbcopy`/`pbpaste` — clipboard copy works with no
extra setup. If `--list-devices` shows no input, grant **Microphone** permission to the
terminal app (System Settings → Privacy & Security → Microphone) — a denied mic surfaces
as an empty device list or an `PortAudioError` at record time, not a Python traceback.

Optional shell alias:

```powershell
function whisper { python "~/.claude\scripts\whisper_dictation.py" @args }   # Windows PowerShell
```

```sh
alias whisper='python3 ~/.claude/scripts/whisper_dictation.py'               # macOS (zsh: ~/.zshrc)
```

## Agent behaviour

1. If the user asks to dictate, run the script (interactive needs a real terminal for Enter-to-stop; prefer `--seconds` when non-interactive).
2. Do **not** invent speech content — only use the script stdout.
3. After transcript lands, treat typos as speech-to-text noise; clarify only if a proper noun is ambiguous.
4. For long `/ask` / research briefs: record → paste → then execute the skill.

## Non-goals

- Not cloud STT; not Mac superwhisper.
- Not always-on hotkey daemon (optional later: AutoHotkey + this script).


_Dr. Mārcis Gasūns_
## References

Detail moved verbatim (read on demand — lives in the rendered skill copy's references/):

- `references/mac-hotkey-lane.md` — Mac hotkey lane — always-on M3 dictation: push-to-talk setup, daemon, permissions, troubleshooting
- `references/change-log.md` — Revision trail
