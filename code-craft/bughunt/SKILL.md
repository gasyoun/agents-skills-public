---
name: bughunt
description: "Run the one-repo bug hunt on demand — the nightly 03:00 routine fired manually, day or night; plus the fix lane for findings from an existing BUGHUNT-FINDINGS report. Triggers: \"/bughunt\", \"/bughunt <repo>\", \"bug hunt\", \"погоняй охоту за багами\", \"fix finding N in <repo>\", \"status bughunt\"."
---

# /bughunt — on-demand one-repo bug hunt + fix lane

## 0. Prior art — consume, don't reinvent (MG 26-09-2026)

GitHub checked BEFORE authoring this skill's methodology (official + user
repos): [anthropics/skills](https://github.com/anthropics/skills) has no
bug-hunt skill; [obra/superpowers](https://github.com/obra/superpowers)
`systematic-debugging`, [doraemonkeys/claude-code-debug-mode](https://github.com/doraemonkeys/claude-code-debug-mode)
and [LerianStudio/ring](https://github.com/LerianStudio/ring) all ship the same
root-cause DEBUGGING discipline — the estate already absorbed it as
`/diagnosing-bugs` (`~/.claude/skills/diagnosing-bugs/SKILL.md`). The nightly
one-repo ROTATION + report landing is estate-owned; the per-bug methodology is
consumed:

- **Found a bug, about to fix it** → load `/diagnosing-bugs` first
  (feedback-loop-first RCA: no fix without a tight red/green loop; the hunt
  FINDS, diagnosing-bugs NAILS the cause).
- **Security-class finding** (secrets, injection, XSS) → `/security-audit`
  methodology for severity + verification, not improvised judgement.
- **After fixing** → verify per `verification-planning` /
  verification-before-completion discipline; changed code → `/code-review`.
- Standing rule: a TYPICAL function → search GitHub FIRST (official repo + 2-3
  user repos) before writing a skill from scratch — MG verbatim: «для типичных
  функций сперва проверяй гитхаб»; author only the estate-specific shell
  (rotation, landing paths, fences). Contract:
  [grok-extra/rules/github-first-external-prior-art.md](https://github.com/gasyoun/claude-config/blob/main/grok-extra/rules/github-first-external-prior-art.md)
  (point 7).

The nightly routine (launchd `com.gasyoun.bughunt-nightly`, 03:00 — MG rulings
26-09-2026: 03:00 · auto-fix HIGH · all repos, giants out) is the same
machinery; this skill fires it interactively. Standing fences: `<client>/`
paths are a hard deny (152-FZ); all repo edits via worktrees off `origin/main`,
PR + merge (always-merge), never a shared main tree; HIGH credential/infra
findings are NEVER self-fixed — GTD `@DO` row instead.

## 1. Status / rotation

```bash
python3 ~/Documents/GitHub/opencode-config/tools/bughunt_night.py --list    # rotation + NEXT
python3 ~/Documents/GitHub/opencode-config/tools/bughunt_night.py --status  # cursor + last runs
```

## 2. Hunt a repo NOW (unattended, night-equivalent)

```bash
python3 ~/Documents/GitHub/opencode-config/tools/bughunt_night.py run --repo <name>
```

Log: `~/.config/opencode/state/bughunt/hunt-<date>-<repo>.log`. Success
advances the cycle cursor — the next 03:00 run picks the following repo.

## 3. Interactive day hunt (MG watching)

When MG asks in chat instead of running the driver: follow the night mission —
read-only focused static hunt of the repo's CODE → ranked findings with
file:line + live evidence → land `BUGHUNT-FINDINGS-<repo>-DD.MM.YYYY.md` at the
repo root via worktree + PR + merge + CHANGELOG bullet. DAY-MODE DIFFERENCE:
MG is present, so MEDIUM findings may be fixed on request — the night autonomy
cap does not bind. Budget ~40 min; stop after 3 failed attempts anywhere.

## 4. Fix findings from an existing report

Reports live at each repo root: `BUGHUNT-FINDINGS-<repo>-<date>.md` (merged).

1. Read the report; MG names finding numbers, or says «fix the HIGHs».
2. Fix each via worktree off `origin/main` + verify with the repo's own gates
   (test suite; else `node --check` / `python3 -m py_compile`) + PR + merge.
3. HIGH credential/infra findings are NEVER self-fixed — mint ONE GTD row:
   `python3 ~/Documents/GitHub/Uprava/tools/gtd_mint_row.py --task "..." --owner MG --tag @DO --link <report blob URL>`.
4. If a fix handoff already exists (`Uprava/handoffs/H###-*.md`), execute it
   via `/go H###` instead of improvising.

Never: force-push, history rewrite, data deletion, credential rotation,
`<client>/` paths, voting (votes are MG's).
