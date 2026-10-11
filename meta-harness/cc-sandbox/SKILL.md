---
name: "cc-sandbox"
description: Run Claude Code (or Gemini CLI) inside the local Docker sandbox with --dangerously-skip-permissions for long unsupervised experiments. Use when the user says "cc sandbox", "container skip-permissions", "sandboxed claude", or needs isolated high-permission agent work.
---

# cc-sandbox

Use this skill when the user asks to run the migrated source command `cc-sandbox`.

## Command Template


# /cc-sandbox [path-to-mount]

_Created: 01-08-2026 · Last updated: 06-09-2026 (H4201 darwin routes)

## Default path

Windows (PowerShell):

```powershell
docker version
cd ~/.claude\container
# if image missing:
docker build -t cc-sandbox .
docker run --rm -it -v "<MOUNT>:/work" -w /work cc-sandbox claude --dangerously-skip-permissions
```

macOS / Linux (sh):

```sh
docker version
cd ~/.claude/container
# if image missing:
docker build -t cc-sandbox .
docker run --rm -it -v "$MOUNT:/work" -w /work cc-sandbox claude --dangerously-skip-permissions
```

`<MOUNT>` = explicit work dir only (default: current project subdirectory the user named).

## Phase 0 — Prereqs

1. `docker version` must succeed. If not: stop and point at [`container/README.md`](https://github.com/gasyoun/claude-config/blob/main/container/README.md) (Docker Desktop install is human).
2. Do **not** invent Docker installation steps that require admin without stating the human must run them.
3. Confirm mount path is not the whole home dir or repo farm unless the user explicitly asked: Windows `<home>` / `Documents\GitHub`; macOS `<home>` / `~/Documents/GitHub`.

## Phase 1 — Build if needed

Windows (PowerShell):

```powershell
docker images cc-sandbox
# empty → docker build -t cc-sandbox ~/.claude\container
```

macOS / Linux (sh):

```sh
docker images cc-sandbox
# empty → docker build -t cc-sandbox ~/.claude/container
```

## Phase 2 — Run

Prefer interactive TTY for first login. For scripted work, write a prompt file into the mount and:

Windows (PowerShell):

```powershell
docker run --rm -v "<MOUNT>:/work" -w /work cc-sandbox `
  claude --dangerously-skip-permissions -p "$(Get-Content prompt.txt -Raw)"
```

macOS / Linux (sh):

```sh
docker run --rm -v "$MOUNT:/work" -w /work cc-sandbox \
  claude --dangerously-skip-permissions -p "$(cat prompt.txt)"
```

(Exact `-p` / headless flags depend on installed Claude Code version — verify with `claude --help` inside the image.)

Gemini CLI variant (when asked): same container, swap the binary — `docker run … cc-sandbox gemini` — and verify its flags with `gemini --help` inside the image; nothing else in this runbook changes.

## Security

- Skip-permissions **only** in this sandbox.
- No secrets in the image; no default mount of `.ssh` or prod keys.
- Money-contour / Systema watcher repos: still use house money + watcher rules; sandbox does not bypass product policy.

## Non-goals

- Not a substitute for worktrees on shared main trees.
- Not for production deploy.

_Dr. Mārcis Gasūns_
