---
name: "cologne-php-xss-sweep"
description: Sweep a Sanskrit Lexicon repo (or org-wide) for reflected-XSS/SQL-injection in its PHP web-frontend and fix with context-correct escaping. PR-only. Security-sensitive - Sonnet 5 (claude-sonnet-5) / Opus 4.8 (claude-opus-4-8) / Fable 5 (claude-fable-5) tier. Do not use for CodeQL/Semgrep alert triage or the broader org security audit — that is /cologne-alert-triage or /cologne-security-audit-all; this fixes PHP reflected-XSS/SQL-injection specifically.

---

# cologne-php-xss-sweep

Use this skill when the user asks to run the migrated source command `cologne-php-xss-sweep`.

## Command Template


# /cologne-php-xss-sweep

_Created: 03-07-2026 · Last updated: 05-09-2026_

Find and fix reflected-XSS, SQL injection, and related injection in the PHP web-interface code of a Sanskrit Lexicon repo, with the **correct escaping for each injection context** (the context, not the function, is what makes a fix right). One tight PR per repo, off `origin/<default>`, never pushing the default branch.

Argument `$ARGUMENTS`: a single repo name (e.g. `csl-santam`), or `all` to enumerate and sweep every php-bearing repo under `gasyoun/*` + `sanskrit-lexicon/*`.

Execute without per-step confirmation. Stop and surface only deeper findings (SQLi needing a refactor, LFI in dead code) per the "Out of scope" rule.

---

## Autonomy & conventions

- **PR-only.** Branch off `origin/<default>`; never push the default branch ([[feedback_autonomous_task_prefs]]). One PR per repo.
- **Resolve the default branch first**: `gh api repos/{owner}/{name} --jq .default_branch` — it is **not always `master`** (mw-dev defaults to `main`; a wrong base makes `gh pr create` fail "No commits between …").
- **Fork gotcha**: some repos (e.g. `csl-websanlexicon` = fork of `funderburkjim/old-websanlexicon`) target the **parent** on `gh pr create` unless you pass `--repo <owner>/<name> --base <default> --head <branch>`. Always pass `--repo`.
- **Parallel-actor dedup**: an external actor runs the same sweep. **Check `origin/<default>` and open PRs FIRST**; if master already has an equivalent fix, do nothing (and CLOSE any duplicate, don't rebase) ([[project_csl_websanlexicon]]).
- **Verify**: `php -l` every changed file with the local XAMPP php (`/c/xampp/php/php.exe`, 8.2.x). Skip Mako-category-`T` template files (`csl-websanlexicon` only) — those aren't standalone PHP.
- Windows: Python scripts must `sys.stdout.reconfigure(encoding='utf-8')` (+ stderr); pass `encoding='utf-8'` to `subprocess.run`; write multi-step scripts to a `.py` file; no PowerShell script-blocks/subexpressions.
- Commit edits via the GitHub **Contents API** (the local clones sit on feature branches/worktrees, so a local `git push` lands on the wrong branch).

---

## 1. Inventory PHP (authoritative)

Use the **git trees API**, NOT `gh search code` (it under-reports and misses forks — it returned empty for `echo $_GET` on repos with 800+ matches):

```sh
gh api "repos/{owner}/{name}/git/trees/<default>?recursive=1" \
  --jq '.tree[].path' | grep '\.php$' | grep -viE 'webbackup/|vendor/|node_modules/'
```

For `all`: enumerate via `gh repo list <owner> --limit 200`, filter to non-dictionary repos with `.php` files, sweep each.

## 2. Detect reflected sinks

`git grep` on the fetched `origin/<default>` ref (not the stale local tree):

```
echo|print|printf|<?=   .*\$_(GET|REQUEST|POST)\[      # raw reflection
\{\$_(GET|REQUEST)\['callback'\]\}                      # JSONP callback
```

Also hunt **stale checked-in display copies** (`vn/*-dev/web/`, `disp/`, `disp1/`) — templates get fixed at source, but per-dict copies committed earlier still echo raw input.

## 3. Fix by injection context (the decision table)

The same input needs **different** escaping depending on where it lands. Pick by sink, not habit:

| Sink context | Example | Correct fix |
|---|---|---|
| HTML text / body | `echo "<h4>$msg</h4>"` | `htmlspecialchars($x, ENT_QUOTES)` |
| HTML attribute | `value="$x"` | `htmlspecialchars($x, ENT_QUOTES)` |
| `<img src="$file">` | image tag | `htmlspecialchars($x, ENT_QUOTES)` |
| JS string inside `<script>` | `displaylink('$x')`, `var a="$x"` | `json_encode($x, JSON_HEX_TAG\|JSON_HEX_AMP\|JSON_HEX_APOS\|JSON_HEX_QUOT)` — **drop the manual quotes**; HEX flags also kill `</script>` |
| **URL-in-JS-string-in-attribute (double context)** | `onclick="f('p=$x')"` | **`urlencode($x)`** — `htmlspecialchars` ALONE fails: the browser HTML-decodes `&#039;`→`'` *before* JS parses, re-enabling breakout; `%27` survives both layers |
| JSONP callback | `echo "{$_GET['callback']}($json)"` | **whitelist** `if(!preg_match('/^[A-Za-z_$][A-Za-z0-9_$.]{0,127}$/',$cb)){http_response_code(400);echo "invalid callback";return/exit;}` then `echo htmlentities($cb)."($json)"` — the whitelist is the real control; `htmlentities` is a no-op over it but **clears the Semgrep taint sink** |
| SQL string literal | `where key='$x'`, SQLite `'$x'` | **prepared statements** + bound params (preferred); SQLite-literal minimal fix = `str_replace("'","''",$x)` |
| SQL `LIMIT` / numeric | `" LIMIT $n"` | `(int)$n` (LIMIT can't be a bound param) |
| SQL table/identifier | `from {$dict}` | whitelist `preg_match('/^[a-z0-9_]+$/',$dict)` (identifiers can't be bound) |
| PCRE pattern (custom `regexp` UDF, `preg_match`) | `preg_match("/$x/")` | `preg_quote($x, '/')` — **a SQLite-literal escape does NOT cover this**; the value reaches a second interpreter (catch the `_sqliteRegexp`-style UDF) |

**Array-injection guard**: `?key[]=x` makes `$_GET['key']` an array. Use `isset()&&is_string()` before `json_encode` — `?? ''` does NOT guard it.

## 4. Known false-positives / no-ops (don't be fooled)

- `filter_var($x, FILTER_UNSAFE_RAW)` (and the deprecated `FILTER_SANITIZE_STRING` replacements) **sanitize nothing** — a `sanitize_*()` function using it is a no-op; fix at the sink.
- A reflection is **safe** when the value is integer-coerced (`(int)$page`, `$page + 14`, `intval()`) or stripped of metachars (`str_replace(['<','>','=','(',')'],'',$x)`), or sourced from a trusted lookup file (e.g. `$filename` from `pdffiles.txt`, not the raw `$_GET`).
- A `workflow_dispatch`/`$_GET` value constrained to a fixed allowlist (e.g. a `type: choice` input, or a `books`-file lookup that `fehler()`-exits on miss) is not injectable.

## 5. Out of scope → surface, don't blind-fix

- **SQLi needing a prepared-statement refactor** of a WHERE-builder, or **LFI** (`fopen($_GET[...])`) in dead `unused_*` files → flag in the PR body / spawn a task; for dead files recommend **deletion**, not a patch.
- Watch for the **second sink**: a SQLi fix often has two (the search term *and* `$maxhits` in LIMIT; the SQL literal *and* the regexp/PCRE context). Don't stop at the first.

## 6. Land it

`php -l` → one squash PR per repo, body naming the sink + the context-correct fix, `Co-Authored-By: Claude Opus 4.8 (1M context)`. After merge, a Semgrep rescan may re-flag the (now-safe) `echo`/`json_encode` of `$_GET` — dismiss those residuals as false-positive-on-fixed-code with a one-line justification.

See [[reference_codeql_no_php]] (php → Semgrep, never CodeQL) and [[project_maintainer_comment_noise]] (keep PR/commit noise minimal). Companion: [`/cologne-alert-triage`](https://github.com/gasyoun/claude-config/blob/main/commands/cologne-alert-triage.md), [`/cologne-security-audit-all`](https://github.com/gasyoun/claude-config/blob/main/commands/cologne-security-audit-all.md).

## Agents (fan-out)

This skill pairs with the read-only agents in [`agents/`](https://github.com/gasyoun/claude-config/blob/main/agents/) (installed at `~/.claude/agents/`):

- **`cologne-security-reviewer`** — for `all`, fan out one per repo to surface candidate sinks (it carries the same context→escaping table).
- **`adversarial-verifier`** — verify each candidate (CONFIRMED / PLAUSIBLE / REFUTED, with the trigger) BEFORE you write the fix. Catches the over-claims and the safe-but-flagged cases.

_Dr. Mārcis Gasūns_
