---
name: code-census
description: "Rank the WORST code across many repos when there is no diff and no fixed point. Triggers: \"what code sucks most?\", \"worst code in my repos\", \"where is the tech debt\", \"code health across the estate\", \"какой код худший\". Produces one ranked, evidence-backed table plus a durable report file. NOT for reviewing a branch, PR, or changes since a ref — that is `code-review`."
---

_Created: 02-09-2026 · Last updated: 05-09-2026_


_Created: 02-09-2026 · Last updated: 02-09-2026_

# code-census — estate-wide worst-code ranking

`code-review` needs a fixed point. This skill exists for the question that has none:
*"what code sucks most?"* over N repos. Without it a session invents the method live —
picking repos ad hoc, re-grepping for the same tooling, polling background scans, and
discovering the privacy fence only after it has already read protected text.

## Phase 0 — Prior art before any scanner is written

```sh
python ~/Documents/GitHub/Uprava/tools/hub_grep.py "code census quality ranking complexity"
```

Name what already exists and reuse it. Only the gap gets new code. A census script written
from scratch when `fruit_report.py`-style machinery already walks the estate is waste.

## Phase 1 — Fence the scope BEFORE reading anything

Decide inclusion/exclusion first, in writing, and apply it as a path filter in the scan
command itself — never as an after-the-fact judgement about text already in context.

- **Privacy fence is mandatory.** Exclude personal-data corpora from the census: transcript
  and client folders (`<client>`, `private`, `personal_data`, payout and payment data).
  152-FZ material is not code health, and pulling it into a ranked report — which is meant
  to be shareable — is the defect this phase prevents.

  ```sh
  --exclude-dir <client> --exclude-dir private --exclude-dir personal_data
  ```

- **Exclude vendored and generated trees** (`node_modules`, `vendor`, `dist`, `.venv`,
  `*_generated.*`) — ranking code nobody owns produces findings nobody can act on.
- **Write the fence into the report** so a reader knows what was NOT looked at.

## Phase 2 — Tooling census, once

Probe once, for all repos, and record the result; do not re-discover per repo:

```sh
for t in ruff pylint radon eslint shellcheck cloc; do command -v "$t" >/dev/null && echo "have $t"; done
```

Missing tooling is a finding of its own ("no linter configured in N of M repos"), not a
blocker. Fall back to language-agnostic signals: file length, function length, duplicate
blocks, churn (`git log --format= --name-only | sort | uniq -c | sort -rn`), and age of the
last touch.

## Phase 3 — Scan in the background, then WAIT

Estate-wide scans exceed a 120s tool timeout. Launch each scan as a background run writing
to a file, then wait for the completion notification.

- **Do not poll.** `sleep`-and-`cat` loops burn a full turn each and produce nothing; the
  harness re-invokes on completion. Schedule one long fallback only if the run may hang.
- **One scan per language family**, launched together, not one repo at a time.
- **Cap the output at the source** (`| head -c 2000000`, `--max-lines`). A 22 MB scan log
  gets persisted and then costs a full re-read every time it is consulted.

## Phase 4 — Rank with evidence, not adjectives

Each ranked entry carries, in one row: repo · path · the measured signal (number, not
"messy") · one quoted line or hunk · the concrete fix. A row without a number and a quote
is an opinion and does not ship.

Rank by **cost to change**, not by ugliness: high churn × high complexity × no tests beats
a long-but-stable file nobody edits.

## Phase 5 — Deliver durably

Write the ranked table to a durable file, not chat only:

```
<repo-or-Uprava>/docs/CODE_CENSUS_<scope>_DD-MM-YYYY.md
```

with the dated header, the fence from Phase 1, the tooling census from Phase 2, the ranked
table, and the exact scan commands so the census is reproducible. Chat carries the top 3
and the link.

## Hard rules

- **No fixed point means no `code-review`.** If the user names a ref, branch, or PR, hand
  off to `code-review` instead — this skill's answer would be strictly worse there.
- **The fence precedes the read.** Privacy exclusions are applied as scan arguments, before
  any protected path is opened.
- **Numbers or nothing.** Every ranked row cites a measurement and a quoted line.
- **Background scans are awaited, never polled.**

_Dr. Mārcis Gasūns_
