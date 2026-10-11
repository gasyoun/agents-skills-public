---
name: "cologne-question-research"
description: Research a question-labeled sanskrit-lexicon issue and post a structured evidence-backed findings comment (never close it yourself) — cross-dictionary, abbreviation, morphology, encoding evidence. Judgment — Sonnet 5 (claude-sonnet-5) tier. Do not use for bug-labeled issues — that is /cologne-bug-triage; this is for question-labeled issues and never closes the issue itself.
---

# cologne-question-research

Use this skill when the user asks to run the migrated source command `cologne-question-research`.

## Command Template


# /cologne-question-research

_Created: 03-07-2026 · Last updated: 05-09-2026_

Research a `question`-labeled issue in the `sanskrit-lexicon` org and post a structured findings comment. Argument: `REPO ISSUE_NUMBER` (e.g. `COLOGNE 272`).

**Goal:** Supply the data that lets a human close the issue. Never close it yourself.

---

## Autonomy rules

- Execute all phases without asking for confirmation.
- All `gh` API calls and `gh issue comment` may be executed freely.
- Windows encoding: all Python scripts include `sys.stdout.reconfigure(encoding='utf-8')`.
- Retry once on transient 5xx/timeouts.

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

Read the full body and all existing comments. If a comment already contains a "Verified" or substantive answer from a non-bot author, post nothing and stop (already answered).

---

## Phase 2 — Classify question type

Based on the body, assign one or more of these types:

| Type | Signals in body |
|---|---|
| `citation` | abbreviation `<ls>`, source name, page number question, "what does X abbreviate?" |
| `morphology` | verb class, root, derivation, paradigm, Whitney, DCS |
| `encoding` | SLP1, IAST, Devanagari, character question, transliteration |
| `editorial-policy` | "should we", "how to represent", "convention", markup style |
| `text-variant` | "print says", "scan shows", "is this correct in source?" |
| `already-done` | body or comments suggest fix already applied |

---

## Phase 3 — Gather evidence

**Delegate the evidence sweep to the read-only `cologne-question-researcher` subagent —
one issue per spawn** (when several question issues are in the backlog, spawn them in
parallel). It runs the whole-tree greps below and returns `VERDICT` + `TYPE` +
`EVIDENCE` (with file:line) + `ANSWER` + `NEXT STEP`; the multi-repo grep spray is
precisely the context this session must not inherit. Phases 3.5–5 — the posting gate,
the comment, the digest row — stay here: the agent never posts. Run the sources below
inline only for a single-issue invocation where a spawn would cost more than the grep.

For each type, check the canonical data source:

### citation
```sh
# Is csl-lslink cloned locally?
ls ~/Documents/GitHub\csl-lslink 2>/dev/null || echo "not cloned"
# Search lslink data for the abbreviation
grep -r "<abbreviation>" ~/Documents/GitHub\csl-lslink\ 2>/dev/null | head -5
# Also check: does the abbreviation appear in csl-orig across dicts?
# grep -r "ABBREV" ~/Documents/GitHub\csl-orig\v02\ --include="*.txt" -l
```

### morphology
```sh
# Check the MW/Whitney root crosswalk (JSON; fields: slp1, class, class_arabic, gloss, mw_L)
ls ~/Documents/GitHub\WhitneyRoots\crosswalk\mw_roots.json 2>/dev/null
# Search for the root by its SLP1 form (substitute ROOT_SLP1, e.g. BU, kf, gam)
grep "\"slp1\": \"ROOT_SLP1\"" ~/Documents/GitHub\WhitneyRoots\crosswalk\mw_roots.json | head -5
```

### encoding
- Check `sanskrit-util` IAST⇄SLP1⇄Devanagari tables (local at `~/Documents/GitHub\sanskrit-util\`).
- Check `SanskritLexicography/FINDINGS.md` for known encoding traps.

### text-variant
```sh
# Find the entry in the source file
grep -n "L>LINENUM\|HEADWORD" ~/Documents/GitHub\csl-orig\v02\DICT\DICT.txt | head -3
```

### editorial-policy
- Read `~/Documents/GitHub\CORRECTIONS\` README and any linked wiki page.
- Check COLOGNE repo wiki if accessible.
- Check `SanskritLexicography/FINDINGS.md` for recorded decisions.

### already-done
- Grep the source for the proposed change — if 0 occurrences, likely already applied.

---

## Phase 3.5 — Gate: only post if there is concrete data

**SKIP the GitHub comment** if the only finding is "Evidence inconclusive — needs human editorial decision" with no supporting numbers, line references, or examples. Inconclusive posts with no data add noise and bury human discussion threads.

**POST** only when at least one is true:
- Assessment is "Answerable: [the answer]" — conclusive finding
- Assessment is "Partially answerable" AND you have concrete evidence (line numbers, counts, specific file locations)
- Assessment is "Appears already resolved" with verifiable proof (grep shows 0 occurrences, etc.)

If the gate is not met: skip posting. Include the finding in your Phase 5 report so the caller can append it to `csl-observatory/docs/question-research-findings.md`.

---

## Phase 4 — Write findings comment

Post exactly one comment wrapped in a collapsible block:

```
<details>
<summary>🤖 Agent research findings</summary>

**Question type:** [citation | morphology | encoding | editorial-policy | text-variant | already-done]

**Evidence found:**
[Concrete data: line numbers, counts, crosswalk hits]

**Assessment:**
[One of:]
- "Answerable: [the answer]"
- "Partially answerable: [partial answer]; [what's still needed]"
- "Appears already resolved: [evidence]"

**Suggested next step:** [one concrete action for the maintainer]

</details>
```

Post via:
```sh
gh issue comment $ISSUE --repo $ORG/$REPO --body "..."
```

---

## Phase 5 — Log

Report back: issue URL, question type, assessment, whether a comment was posted, and (if skipped) the key finding for the digest file.

_Dr. Mārcis Gasūns_
