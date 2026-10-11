---
name: "prior-art"
description: 'Run the check-prior-art escalation before building anything — memory → current repo → SHARED_CODE/sanskrit-util → sibling repos → web — report "exists at X, build only the gap". BEFORE any "let me write a parser/helper for Y" or "check prior art". Also: "/prior-art", "does this already exist", "don''t reinvent the wheel", "has this been built before".'

---

# prior-art

Use this skill when the user asks to run the migrated source command `prior-art`.

## Command Template


# /prior-art

_Created: 03-07-2026 · Last updated: 05-09-2026_

## Default path (seconds — run this first)

```sh
python ~/Documents/GitHub/Uprava/tools/hub_grep.py "<query>"          # OR-match, ranked, 20 hits
python ~/Documents/GitHub/Uprava/tools/hub_grep.py --all -m 50 "<query>"   # every term must hit; more hits
```

[`hub_grep.py`](https://github.com/gasyoun/Uprava/blob/main/tools/hub_grep.py) (flags: `--all`, `-m/--max N`, `--no-external`, `--selftest`; local-only, no network) scans every manifest hub, the whole `Uprava/handoffs/` tree (active + archive), both FINDINGS registries and the read-only `GitHub-external/` clones — i.e. ladder levels 3–4 and the handoff registry in one call. It does **not** scan `MEMORY.md` or the target repo's own git history: levels 1–2 stay manual.

LLM / judgment residue only after the script report exists (or the script exits non-zero with a real defect).

The standing rule made one command: checking is far cheaper than rebuilding, and most tasks here repeat across ~250 local clones. Stop at the **first real hit** — do not run all levels for completeness.

Argument: one line describing what you are about to build or derive (or infer it from the current conversation).

## Escalation ladder — cheapest first, stop at first hit

1. **Memory index** — scan `MEMORY.md` for the concern; open the matching `project_*` / `reference_*` file. Many assets are indexed there precisely because they keep being rebuilt (corpus_lexicon, union_headwords, glossary, crosswalks).
2. **Current repo — `git fetch origin` FIRST, then read.** A local clone can be stale by days/dozens of commits with no visible signal (the [H919 (Sonnet 5) — SamudraManthanam НКРЯ wave-3 sanskritisms layer](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H919-Sonnet_SamudraManthanam_nkrya-wave3-sanskritisms-layer_14.07.26.md) incident, 14-07-2026: a SamudraManthanam session's `.ai_state.md` read "W3 queued" from disk while `origin/main` — 2 days and ~40 commits ahead — had already shipped that exact work and closed it; the session built a full duplicate package before its first `git fetch` of the session surfaced the collision). `git fetch origin && git log --oneline origin/main -10` (or `git log --oneline HEAD..origin/main` to see exactly what's missing) is not optional preamble — do it before trusting anything else in this step. Only then: targeted `Glob` of the relevant subdir (NEVER a repo-wide glob from the GitHub root — it times out), plus `.ai_state.md`, `README`, `docs/`, and — since a repo can label work with its own informal `H###` in commit messages/`.ai_state.md` without ever claiming it via `mint_handoff.py` (making it invisible to Uprava's registry by construction) — a quick `git log --oneline --all --grep=<topic>` in the target repo catches what the registry alone cannot.
3. **Org hubs** — [`github-spine/SHARED_CODE.md`](https://github.com/gasyoun/github-spine/blob/main/SHARED_CODE.md) (9+ canonical code families — transcoders, normalizers, correction appliers, dict-web endpoints), [`Uprava/PROJECT_INTERLINKS.md`](https://github.com/gasyoun/Uprava/blob/main/PROJECT_INTERLINKS.md) (derived data feeds: alignments, frequency tables, headword/root indexes, OCR, morphology), [`sanskrit-util`](https://github.com/sanskrit-lexicon/sanskrit-util) (Py+JS transcoding). Also both FINDINGS registries for measured caveats about the thing — [Uprava/FINDINGS.md](https://github.com/gasyoun/Uprava/blob/main/FINDINGS.md) (infra/process) and [SanskritLexicography/FINDINGS.md](https://github.com/gasyoun/SanskritLexicography/blob/master/FINDINGS.md) (Sanskrit data).
4. **Sibling / org repos** — the repo that *owns* the concern per PROJECT_INTERLINKS; `gh search code --owner sanskrit-lexicon <term>` when ownership is unclear.
5. **GitHub / web at large** — for a general problem (transcoder, alignment algorithm, format parser, known research method), search for a maintained open-source implementation or published approach. Adapt, don't reinvent; cite what you found.

## Verdict — always one of three

- **EXISTS** — name the canonical source with a clickable link; consume it, build nothing.
- **PARTIAL** — name what exists, then state the *gap only* as the build scope.
- **NEW** — say which levels were checked (so the next session doesn't re-check), then build.

If the verdict is EXISTS or PARTIAL and the discovery took real effort, append a pointer to the right hub per [`/findings-append`](https://github.com/gasyoun/claude-config/blob/main/commands/findings-append.md) so the next chat finds it at level 1–3.

_Dr. Mārcis Gasūns_

