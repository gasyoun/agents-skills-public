---
name: "context"
description: 'Build a sourced context pack for an unfamiliar handoff. Use only when explicitly invoked as /context.'
---

# context

Use this skill when the user asks to run the migrated source command `context`.

## Command Template


# /context

_Created: 20-07-2026 · Last updated: 13-09-2026_

## Default path (seconds — run this first)

**1. Hybrid hub recall first** (H2447 / H406–H407) — ranked notes before lexical greps:

- Prefer MCP `uprava-recall` → tool `memory_recall` (hybrid default, `k` 5–8).
- **Pure token queries go bm25-first** (H4618, 12-09-2026): when the ask is an
  exact ID/code/ref (`H4618`, `csl-orig`, an error slug) pass
  `strategy="bm25"` to `memory_recall` (or `--strategy bm25` on the CLI) —
  literal word overlap beats hybrid fusion on exact tokens.
- CLI fallback:
  ```sh
  python ~/Documents/GitHub/Uprava/tools/recall_index.py --strategy hybrid -k 5 "<task paraphrase>"
  ```
- Install / harness inventory: [RECALL_MCP_INSTALL.md](https://github.com/gasyoun/Uprava/blob/main/tools/RECALL_MCP_INSTALL.md).

**2. Lexical hub sweep** (always useful for exact registry rows):

```sh
python ~/Documents/GitHub/Uprava/tools/hub_grep.py "<query>"
python ~/Documents/GitHub/Uprava/tools/hub_grep.py --selftest
```

LLM / judgment residue only after at least one of those reports exists (or a script
exits non-zero with a real defect). Same retrieval order as `/semantic-memory search`:
`memory_recall` → GrayMatter (if available) → `hub_grep`.

The CONSUME half of the org's registry machinery (H1393, wave-1 lane 2 of
[PLAN_CROSS_REPO_REUSE_CONTEXT_2026Q3.md](https://github.com/gasyoun/Uprava/blob/main/PLAN_CROSS_REPO_REUSE_CONTEXT_2026Q3.md)):
the hubs are dense, but nothing else brings the task-relevant slice INTO a session.
Input: a one-line task description (argument, or infer from the conversation). Output:
one **context pack** message, every item carrying its source link. This skill only
READS — no writes, no network beyond `git fetch` freshness checks.

## Procedure

1. **Tokenize the task.** Pick 3–6 discriminative tokens: the repo/subsystem name, the
   asset (e.g. `corpus_lexicon`, `TMX`, `digentry`), the operation (`fuzzy`, `sandhi`,
   `probe`), plus obvious synonyms. Skip stop-words; prefer tokens likely to appear in
   hub rows. Also keep a **one-line natural-language paraphrase** of the task for
   `memory_recall` (paraphrases beat keyword-only BM25 on the hybrid tier).

2. **Recall, then sweep the hubs.**

   a. **Hybrid recall** (step 0 / default path above) with the paraphrase.

   b. For each token set, run lexical greps (from any repo under `GitHub/`):

   ```sh
   python ../Uprava/tools/hub_grep.py -m 30 "<token1>" "<token2>" ...
   python ../Uprava/tools/hub_grep.py --all -m 15 "<token1>" "<token2>"   # tighter pass
   ```

   Adjust the relative path to the Uprava clone (`../Uprava` from a repo root;
   `tools/hub_grep.py` inside Uprava itself). The tool covers every hub row in
   [`tools/hub_manifest.json`](https://github.com/gasyoun/Uprava/blob/main/tools/hub_manifest.json)
   (**44** rows as of 03-09-2026 — recount from the manifest rather than trusting this
   number), the whole `handoffs/` + `handoffs/archive/` tree, both FINDINGS families,
   [SHADOW_ASSETS_POINTERS.md](https://github.com/gasyoun/Uprava/blob/main/SHADOW_ASSETS_POINTERS.md)
   (through its ordinary manifest row), and the `Documents/GitHub-external/` adjacent-org
   read-only clones.

   **The external-mirror staleness check is yours, not the tool's.** `hub_grep.py` has no
   staleness reporting — it never reads or mentions `CLONES_STAMP.json` — and when
   `Documents/GitHub-external/` is absent (it is on some boxes, including this Mac) it
   scans nothing there **silently**, so "nothing shipped upstream" and "no mirrors
   installed" look identical in the output. Read the stamp yourself, no network:

   ```sh
   python ../Uprava/tools/refresh_external_clones.py --list
   ```

   Stamp older than 7 days ⇒ say "external mirrors stale (DD-MM-YYYY)" in the pack; no
   stamp / no external root ⇒ say "external mirrors absent — upstream coverage unchecked".
   Either way offer `python ../Uprava/tools/refresh_external_clones.py`.

3. **Pull the danger facts.** Read the target repo's section of
   [DANGER_FACTS.md](https://github.com/gasyoun/Uprava/blob/main/DANGER_FACTS.md) (or
   [context_facts.yaml](https://github.com/gasyoun/Uprava/blob/main/context_facts.yaml)
   for the machine-readable form) + the Org-wide section. These go in the pack even if
   no query token matched them — a session that misses one destroys work.

4. **Pull the edges.** From
   [interlinks_edges.tsv](https://github.com/gasyoun/Uprava/blob/main/interlinks_edges.tsv):
   rows where the target repo is `source` or `target` — what this repo consumes,
   produces, and what is queued/proposed around it.

5. **Prior handoffs.** From the hub_grep hits, keep the 2–5 most relevant `H###` bodies
   (active before archive) and read their goal/stop-condition lines — the fastest
   "has someone shaped this exact task before" signal. Note any 🔒 CLAIMED /
   in-flight sibling.

6. **Emit the pack.** One message, this shape, every line linked:

   > **Context pack — "<task>"**
   > - **Hybrid recall:** <top memory_recall hits — ref + one-line each, or none>
   > - **Owning hubs:** <hub row hits, 3–6 lines max>
   > - **Canonical assets:** <asset → owner → how to consume; shadow-registry rows for off-git bytes>
   > - **Danger facts:** <repo section + org-wide, verbatim-short>
   > - **Edges:** <consumes/produces/queued>
   > - **Prior handoffs:** <H### — one-line each, status>
   > - **Staleness notes:** <external-mirror stamp, behind-origin clones, anything (unverified)>

   Cap the pack at ~40 lines — it is a routing artifact, not a report. If a section is
   empty, say `none found` (that absence is itself signal).

## Quality bar

- ≥3 distinct hubs cited for any real task, including ≥1 handoff body — if fewer hit,
  say so and broaden tokens once before concluding "thin coverage".
- Never paste bytes from rights-gated/shadow assets — pointer rows only.
- The pack must distinguish MEASURED facts (linked) from inferences ("(unverified)").

## Compatibility Rule

Write and follow this skill so it works in both Codex and Claude Code. Treat
`/skill-name` references as workflow names: in Codex invoke the skill when available;
in Claude Code read the named command file and follow it manually.

_Dr. Mārcis Gasūns_
