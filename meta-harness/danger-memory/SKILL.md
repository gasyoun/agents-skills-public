---
name: "danger-memory"
description: 'Persist a destructive-risk fact (wiping seeder, script that must never re-run, work-destroying watcher) to the repo''s OWN memory store + org FINDINGS, then sync the ~/.claude backup — one command. Triggers: "danger-memory", "remember this danger", "never run this again", "record a destructive-risk fact". Judgment — Sonnet 5 (claude-sonnet-5) tier.'
---

# danger-memory

Use this skill when the user asks to run the migrated source command `danger-memory`.

## Command Template


# /danger-memory

_Created: 14-07-2026 · Last updated: 04-10-2026_

The global CLAUDE.md **"Memory routing — dangerous facts go to the repo's OWN store"** ritual, made one command. Persistent memory loads by the directory a session STARTS in, so the rich org-level store is invisible to any session cd'd directly into a repo. A destructive-risk fact learned in-repo therefore has to be written to **that repo's own store** or the next in-repo session never sees it — and then mirrored up and the config backup pushed. This skill does all three legs so none is skipped.

Argument: a one-line description of the danger fact (or infer it from the current conversation).

The finding-routing twin is [`/findings-append`](https://github.com/gasyoun/claude-config/blob/main/commands/findings-append.md) — that routes ordinary gotchas/datasets/taxonomies into hub docs; **this** handles the narrow, higher-stakes class of *destructive-risk* facts that must reach the per-repo memory store. When a fact is both a normal caveat AND destructive-risk, run both.

---

## When to use — the destructive-risk gate

Fire ONLY for a fact where acting without it destroys work or data. Canonical classes (from the rule):

- a **seeder / import that wipes human overlays** (re-running it clobbers hand-curated data);
- a **script that must never be re-run** (idempotency-breaking, one-shot migration);
- a **watcher / sync client that destroys uncommitted work** (reverts the tree, autostash-swallows a concurrent session);
- a **branch-contention actor** (a bot/second session that commits to your branch, a `git pull --autostash` / `git stash -u` that swallows another tree).

**When NOT to use:** an ordinary, non-dangerous fact (an encoding quirk, a path, a measured count, a taxonomy) — that is normal memory / a hub finding → [`/findings-append`](https://github.com/gasyoun/claude-config/blob/main/commands/findings-append.md), not this. If nothing gets destroyed by not knowing it, it is not a danger fact.

---

## Phase 0 — Locate the two stores

Memory stores live under a **projects root**, and *which* root depends on the layout this box runs — get this wrong and every later phase writes into a directory nothing reads:

| Layout | Projects root | Slug shape (a real one on that box) |
|---|---|---|
| **default (this Mac, and any plain install)** | `~/.claude/projects/` | `-Users-mac-Documents-GitHub-<Repo>` — leading `-`, no drive letter |
| per-profile (the `<claude-tools>/profiles/claude1/` estate) | `<profile>/.claude/projects/` | `C--Users-user-Documents-GitHub-<Repo>` — drive-colon flattened to `--` |

In both, the slug is the session's start directory with every separator (and the drive colon, where there is one) turned into `-`. Both forms coexist under `~/.claude/projects/` today, so **`ls` the root and match an existing sibling's shape — never hand-derive the slug and hope**.

- **repo's own store** — `<projects-root>/<slug-of-that-repo>/memory/` (e.g. `…-SanskritLexicography/memory/`). **Create the `memory/` dir + a `MEMORY.md` index if absent** — this is the leg the audit found missing (danger facts sat only in the org store).
- **org store** — `<projects-root>/<slug-of-GitHub-root>/memory/` — visible to sessions that start in `GitHub/` but NOT to ones cd'd into a single repo.

**Worktree sessions: never write to your own store.** Estate rails put this session in a `<repo>-h<id>-<pid>` worktree, so its slug is the *worktree* path — a directory deleted at the end of the pass, taking the danger fact with it. [`hooks/sessionstart_context.py`](https://github.com/gasyoun/claude-config/blob/main/hooks/sessionstart_context.py) prints the H1797 bridge naming the **canonical** main-tree store to write to instead (tested by [`hooks/test_sessionstart_memory_bridge.py`](https://github.com/gasyoun/claude-config/blob/main/hooks/test_sessionstart_memory_bridge.py)); use that path. An abandoned worktree store on disk is what this looks like when it is skipped.

Confirm both paths with `ls` before writing. Read the repo store's `MEMORY.md` to match its existing index style.

## Phase 1 — Write the danger fact to the repo's OWN store

Create `<repo-store>/memory/<kebab-slug>.md`. Slug names the hazard, not the session (`git-stash-u-swallows-concurrent-tree`, `pwg-ru-store-worktree-persistence`). Frontmatter matches the sibling memory files exactly:

```
---
name: <kebab-slug>
description: "<one sentence naming the hazard AND the safe alternative — the danger stated plainly>"
metadata:
  node_type: memory
  type: feedback
  originSessionId: <this session id>
---

<body: what the danger is · WHY it happens (the mechanism) · the exact trigger to avoid ·
the safe alternative / recovery command · date + repo + handoff (H###) it was learned in ·
every path as a full blob URL>
```

`type:` is `feedback` for a behavioural/danger hazard (`project` / `reference` are for the non-danger classes). Then add ONE index line to that store's `MEMORY.md`:

```
- <title> — <danger + safe-alternative in one line, load-bearing detail kept>.
```

## Phase 2 — Mirror up: org store and/or FINDINGS

The fact now lives where an in-repo session sees it; also put it where a `GitHub/`-level session or a human browsing hubs will find it. Pick by reach:

- **Org memory store** (`…-GitHub/memory/`) — if the hazard could bite from a session that starts at the org root, or spans repos (a branch-contention actor, a shared-tree swallow). Same frontmatter + one `MEMORY.md` index line.
- **FINDINGS hub** — the durable, human-readable mirror. Infra/git/watcher hazard → [Uprava/FINDINGS.md](https://github.com/gasyoun/Uprava/blob/main/FINDINGS.md); Sanskrit-data destructive hazard (a seeder wiping dictionary overlays) → [SanskritLexicography/FINDINGS.md](https://github.com/gasyoun/SanskritLexicography/blob/master/FINDINGS.md). Dedupe first (grep the key terms); if an entry exists, append a dated addendum instead of a near-duplicate.

At least ONE mirror is mandatory — a danger fact in the repo store alone still repeats the exact failure the audit named (a fact reachable from only one entry point).

**MOC pointer (added 04-10-2026, H5808).** The mirrors above are the warehouse; the [SecondBrain-LLM-Wiki](https://github.com/gasyoun/SecondBrain-LLM-Wiki) knowledge map carries the pointer too, or «что я знаю про X» still needs cross-repo grep. Add ONE row for the danger fact to the matching manual MOC (`10-Projects/Uprava-index.md` / `20-Areas/Ops-index.md` / `30-Resources/Doctrine-index.md` / `10-Projects/Sanskrit-index.md`): `[[atomic-note]] — hazard in six words + the recovery command + date`. Never the generated `40-Estate/*-MOC.md` layer — the weekly puller rewrites those wholly (vault rule 3) and a hand row there is wiped; if only a generated MOC matches, record the gap in the closest manual MOC instead, and when no manual MOC stream fits at all, in the hazard's project note «Журнал» (failing that a one-liner under `Home.md`'s streams list) — hand-editable, never the auto layer. Full routing contract: [`/findings-append` Phase 6](https://github.com/gasyoun/claude-config/blob/main/commands/findings-append.md).

## Phase 3 — Sync the ~/.claude backup

Immediately after writing memory, push the config backup (the standing rule — do NOT defer):

```sh
cd ~/.claude && git add -A && git commit -m "backup: config sync" ; git push
```

Private repo [gasyoun/claude-config](https://github.com/gasyoun/claude-config). The `;` before `git push` is load-bearing: with nothing staged, `git commit` exits **non-zero**, and an `&&` there would skip the push and strand whatever drift was *already* unpushed — the precise outcome this phase exists to prevent. A no-op commit is fine; a skipped push is not.

## Phase 4 — Commit the FINDINGS mirror

If Phase 2 touched a FINDINGS hub, commit it in its own repo (`docs:` prefix) and push — [Uprava](https://github.com/gasyoun/Uprava) and [SanskritLexicography](https://github.com/gasyoun/SanskritLexicography) both take direct commits on their default branch.

**Whether the memory `.md` files themselves need committing depends on the layout from Phase 0** — and under the default one they do, which is why Phase 3 is not optional:

- **Default layout.** `~/.claude` *is* the `claude-config` working tree, and its [`.gitignore`](https://github.com/gasyoun/claude-config/blob/main/.gitignore) deliberately whitelists the stores out of the otherwise-ignored `projects/` tree (`!/projects/*/memory/`, `!/projects/*/memory/**`) — 327 memory files are tracked as of 03-09-2026. So the file just written **is** version-controlled, and Phase 3's `git add -A` is exactly what commits it. Treating Phase 3 as a cosmetic "backup" here leaves the danger fact uncommitted on one machine.
- **Per-profile layout.** Those stores really do sit outside the repo (different drive/profile tree), so the `projects/` whitelist never reaches them. They are not orphaned either: [`scripts/memory_backup_sync.py`](https://github.com/gasyoun/claude-config/blob/main/scripts/memory_backup_sync.py) mirrors every profile's `*.md` into the tracked `memory-backup/` directory (`--dry-run` to preview, `--no-push` to commit locally). Run it in the same pass — that is the third leg for this layout.

## Safety notes

- **Never soften a danger fact into a neutral note.** State plainly what gets destroyed and the one command that avoids/recovers it — a hedged memory is worse than none because it invites the risky action.
- **Recovery command belongs in the body.** If the hazard has a known recovery (reflog→reset→pop for a stash swallow; `/branch-contention-recover` for a foreign commit), name it — the next session hits the hazard mid-incident and needs the fix, not just the warning.
- Do not run destructive git here (no force-push, no history rewrite); Phase 3/4 are ordinary `add`/`commit`/`push` on the current branch.

## Output

Report: the repo-store file written (new/updated), the mirror(s) touched (org store · which FINDINGS), the MOC pointer row (which manual MOC, or the gap note), the index lines added, and the commit/push SHAs for the backup sync and any FINDINGS commit.

_Dr. Mārcis Gasūns_
