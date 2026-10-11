---
name: "cologne-bug-triage"
description: Triage one bug-labeled sanskrit-lexicon issue — classify agent-fixable (Tier A) vs needs-server/human (Tier D), gather concrete source evidence, post a triage comment only when concrete. Judgment — Sonnet 5 (claude-sonnet-5) tier. Do not use for question-labeled issues — that is /cologne-question-research; do not use for full-repo issue-taxonomy labeling — that is /cologne-issue-runbook.
---

# cologne-bug-triage

Use this skill when the user asks to run the migrated source command `cologne-bug-triage`.

## Command Template


Triage a `bug`-labeled open issue in the `sanskrit-lexicon` org. Argument: `REPO ISSUE_NUMBER` (e.g. `csl-orig 2811`).

_Created: 03-07-2026 · Last updated: 05-09-2026_

**Goal:** Classify the bug, check whether it is agent-fixable (Tier A) or needs server/rendering/human (Tier D), and supply the data that lets a maintainer act. Never close issues. Never post if the only finding is "inconclusive."

---

## Autonomy rules

- Execute all phases without asking for confirmation.
- All `gh` API calls and `gh issue comment` may be executed freely.
- Windows encoding: all Python scripts include `sys.stdout.reconfigure(encoding='utf-8')`.
- Retry once on transient 5xx/timeouts.
- **Do NOT post** if the only finding is "inconclusive" or "needs human judgment" with no supporting data.

---

## Phase 0 — Parse arguments

```
ORG=sanskrit-lexicon
REPO=<first word of $ARGUMENTS>
ISSUE=<second word of $ARGUMENTS>
```

---

## Phase 1 — Fetch issue

```sh
gh issue view $ISSUE --repo $ORG/$REPO --json number,title,body,labels,comments
```

Read body and ALL comments. If a non-bot author has already posted a concrete resolution or "fixed in PR#N", post nothing and stop.

---

## Phase 2 — Classify bug type

| Class | Signals | Agent action |
|---|---|---|
| `source-error` | Wrong text/markup in a `.txt` source file; exact lines cited or greppable | Tier A: write fix, open PR |
| `xml-error` | Malformed XML tag, unclosed element, bad nesting; verifiable by grep | Tier A: write fix, open PR |
| `broken-link` | Dead URL, wrong `href`, anchor no longer exists | Tier A: fix URL in source/config |
| `download-broken` | ZIP/PDF download 404; server-side file missing | Tier D: needs server check |
| `display-rendering` | Wrong output in browser; correct in source | Tier D: needs websanlexicon/csl-app |
| `search-broken` | Search returns wrong results; correct in data | Tier D: needs server |
| `encoding-display` | Character shows wrong in UI; correct in source SLP1 | Tier D: needs transcoder/display |
| `already-fixed` | Grep shows 0 occurrences of the problem, or PR already merged | Post "appears already fixed" with evidence |

---

## Phase 3 — Gather evidence

**Delegate the diagnosis to the read-only `cologne-bug-diagnoser` subagent — one issue
per spawn**, and in parallel when [`/issue-dispatch`](https://github.com/gasyoun/claude-config/blob/main/commands/issue-dispatch.md) routes a batch
of bug issues here. It returns `VERDICT` (bug class) + `TIER` + `EVIDENCE` +
`PROPOSED FIX` + `TIER D OWNER`; reading every comment then grepping the dictionary
source and fetching file contents is the per-issue context this session must not inherit.
Phases 3.5–5 — the gate, the comment, and routing any Tier A fix through
[`/cologne-correction-queue`](https://github.com/gasyoun/claude-config/blob/main/commands/cologne-correction-queue.md) — stay here.

### For source-error / xml-error
```powershell
# Find the dict source
$src = "~/Documents/GitHub\csl-orig\v02\DICT\DICT.txt"
Select-String -Path $src -Pattern "PATTERN" | Select-Object -First 10
```

### For broken-link
```sh
gh api repos/$ORG/$REPO/contents/FILE --jq '.content' | python -c "import sys,base64; sys.stdout.reconfigure(encoding='utf-8'); print(base64.b64decode(sys.stdin.read()).decode())"
# Then grep for the dead URL
```

### For already-fixed check
```powershell
Select-String -Path $src -Pattern "BROKEN_PATTERN" | Measure-Object | Select-Object -ExpandProperty Count
# 0 hits → already fixed
```

### For display-rendering / search / download
- Read the issue body carefully for the exact URL or search term
- Note whether the source file is correct (grep confirms) or also wrong
- Classify as Tier D with a note: "source is correct at line N; rendering layer needs investigation"

---

## Phase 3.5 — Gate: only post if concrete data

**SKIP posting** if:
- Only finding is "this needs server access" with no data
- Already extensively discussed (5+ non-bot comments with no open question)
- Finding is identical to an existing comment

**POST** if:
- You found the bug in source (line number, count, excerpt)
- You confirmed it is already fixed (grep = 0 hits)
- You can pinpoint the display/config layer that owns the fix (file + line)
- Assessment is "Tier A fixable" — post the proposed fix directly

---

## Phase 4 — Write triage comment

```
<details>
<summary>🤖 Bug triage findings</summary>

**Bug class:** [source-error | xml-error | broken-link | display-rendering | search-broken | encoding-display | already-fixed | download-broken]

**Evidence:**
[Concrete: line numbers, grep counts, URL status, file location]

**Agent-fixable (Tier A)?** [Yes — proposed fix below | No — needs server/display layer]

**Proposed fix (if Tier A):**
```
[change file lines or diff]
```

**If Tier D — what needs checking:**
[exact file/component + what to look for]

**Suggested next step:** [one action]

</details>
```

Post via:
```sh
gh issue comment $ISSUE --repo $ORG/$REPO --body "..."
```

---

## Phase 5 — Log

Report: issue URL, bug class, Tier A/D, and whether a comment was posted.

_Dr. Mārcis Gasūns_
