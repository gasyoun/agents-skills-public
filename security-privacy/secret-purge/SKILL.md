---
name: "secret-purge"
description: 'Purge a leaked secret from a repo''s git history — rotate first, git filter-repo, coordinated force-push, merge-ref verification. DESTRUCTIVE, gated on explicit human confirmation. Triggers: purge the key from history, secret-scanning fired.'

---

# secret-purge

Use this skill when the user asks to run the migrated source command `secret-purge`.

## Command Template


# /secret-purge

_Created: 03-07-2026 · Last updated: 26-08-2026_

History rewriting is destructive, rare, and easy to get wrong. Precedents: RuWritingStyles corpus split + purge (force-pushed 2026-06-12) and the IndologyScholars deepseek key smear. Follow the order exactly — the order IS the safety.

Argument: `<repo> <secret-or-path-pattern>`.

## Phase 0 — Two gates (both mandatory)

1. **Rotate/revoke FIRST.** Purging does not un-leak: the secret lives in clones, forks, caches, and scanners' archives. Confirm the credential is rotated or revoked (a human @DO if it needs an account login — add to GTD and STOP until done). For rights/privacy content there is no rotation — note that removal limits future exposure only.
2. **Explicit human confirmation** of the rewrite, in this session, after showing the blast radius: number of commits rewritten, open PRs that will orphan, forks that keep the old history, collaborators who must re-clone. Never proceed on an old or implied approval.

## Phase 1 — Inventory the exposure

- All occurrences in history: `git log -S'<secret>' --all --oneline` (or by path).
- GitHub-side copies the rewrite will NOT remove: PR diffs, **merge-ref code-scanning/secret-scanning alerts** (the IndologyScholars lesson — check alerts on merge refs, not just branches), issue/comment quotes, releases, forks.

## Phase 2 — Rewrite

Use `git filter-repo` (never `filter-branch`) on a **fresh clone**: `--replace-text` for a string secret, `--invert-paths --path <file>` for a file. Verify the secret is gone: re-run the `-S` search over the rewritten history; check the repo still builds.

## Phase 3 — Coordinated force-push

Temporarily lift branch protection if it blocks the push (re-enable immediately after — the protection baseline comes from the [`/cologne-deploy-all`](https://github.com/gasyoun/claude-config/blob/main/commands/cologne-deploy-all.md) `branch-protect` rung, which replaced the retired `cologne-branch-protect-all.md`). `git push --force --all && git push --force --tags`. Ask GitHub Support / use the UI to dismiss cached views if the leak was sensitive; contact fork owners or note forks as residual risk.

## Phase 4 — Verify + close out

- Secret-scanning / code-scanning alerts: resolve or dismiss with written justification; verify none remain on merge refs.
- Fresh clone → confirm absence; collaborators/CI re-clone note.
- Document the incident (what leaked, when, rotation time, rewrite SHA range) in the repo's `.ai_state.md` and append the pattern to `Uprava/FINDINGS.md`. Add any residual @WAITING (support ticket, fork owners) to GTD.

_Dr. Mārcis Gasūns_
