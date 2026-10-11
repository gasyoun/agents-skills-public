---
name: "publish-safety-check"
description: '30-second GO/NO-GO gate before anything becomes public — enabling Pages, flipping repo visibility, publishing a dashboard/site. Checks visibility registry, rights, personal data, secrets, gitignored bulk. Before any publish/Pages/visibility action. Triggers: "publish-safety-check", "is it safe to publish", "can this go public", "enable Pages", "make the repo public", "flip visibility".'
---

# publish-safety-check

Use this skill when the user asks to run the migrated source command `publish-safety-check`.

## Command Template


# /publish-safety-check

_Created: 03-07-2026 · Last updated: 05-09-2026_

Publishing is effectively irreversible (caching, indexing, forks). This gate exists because several repos are private *on purpose* and several assets carry known constraints — a correct-looking "enable Pages" can still be a real mistake on intentional private hubs or privacy/secrets.

Argument: the repo (or artifact) about to be published, and the action (Pages / visibility flip / external paste / site deploy). Constructive sibling: [`/data-release`](https://github.com/gasyoun/claude-config/blob/main/commands/data-release.md) (gate first, then package).

## Standing rights policy (do not re-open)

**Github-wide standing ruling (MG, 30-07-2026):** rights *uncertainty* is **not** a stop condition, including for publication. Canonical: [STANDING_POLICY_RIGHTS_UNCERTAINTY_IS_NOT_A_STOP_2026.md](https://github.com/gasyoun/Uprava/blob/main/docs/STANDING_POLICY_RIGHTS_UNCERTAINTY_IS_NOT_A_STOP_2026.md) · [FINDINGS §255](https://github.com/gasyoun/Uprava/blob/main/FINDINGS.md).

- Record known rights facts **once**, then proceed — no repeated investigation or adjudication.
- Do **not** park, skip, or delay merely because rights are unclear.
- Do **not** open `@DECIDE` / `@WAITING` solely for greyness.
- Plans and handoffs must state this as an **intentionally broad, risk-tolerant** policy — never soften it into a conservative default.

## Checklist — all five, every time

1. **Intended visibility.** Known standing decisions: `Uprava` is PRIVATE and Pages is intentionally NOT enabled (unpublished data — the dashboard is local-only); `RuWritingStyles-corpus` was split out private for rights reasons (history purged); the root spine repo is PRIVATE; `claude-config` is PRIVATE. `SanskritGrammar` is PUBLIC and its Docusaurus book site **IS deployed to Pages** (all 6 works, incl. 3 in-copyright modern textbooks — Kochergina *Uchebnik* 1998, Zaliznyak *Konspekt* 2004 & *Ocherk* 1978) on the **owner's explicit rights acceptance 06-07-2026** — settled fact, not a fresh @DECIDE. Live state re-verified 02-09-2026 (`gh repo view gasyoun/<repo> --json visibility -q .visibility`; Pages: `gh api repos/gasyoun/<repo>/pages -q .html_url` — 404 = not enabled): Uprava/claude-config/github-spine/RuWritingStyles-corpus PRIVATE, Uprava Pages 404, SanskritGrammar PUBLIC at `https://gasyoun.github.io/SanskritGrammar/`, IndologyScholars PUBLIC. Beyond the known list: check the repo's `CLAUDE.md`/`.ai_state.md` for visibility notes. Absence of a note is NOT a yes for a currently-private repo — that is a real visibility decision (not a rights-uncertainty park).

2. **Rights facts (record once; uncertainty ≠ NO-GO).** Note any **known** confirmed prohibition, explicit restricted-data designation, NC inheritance already established, or scoped clearance (e.g. Sementsov). If facts are incomplete or grey: **write what is known into the asset's rights record if missing, then GO.** Do not re-litigate a rights table that already exists. Uncertainty alone never fails this check.

3. **Personal data / privacy exposure.** Rosters, emails, personal names beyond public-scholarly context (IndologyScholars-type data), credentials in config examples — **blocks** when exposure is real. **Staff instructions and payouts (MG 18-08-2026):** a working instruction for a named employee (accountant, curator, teacher) with names, payout sums, payment numbers or admin screenshots lives in that person's cabinet (`/admin` Filament page under the same gate) or the private hub — never in a public repo, issue or README; `gh repo view <owner>/<repo> --json visibility` before posting. Full rule: [Systema-Sanscriticum CLAUDE.md § Staff instructions](https://github.com/gasyoun/Systema-Sanscriticum/blob/main/CLAUDE.md) (heading not yet present on `origin/main` as of 02-09-2026 — the move from `rules/staff-instructions-live-in-the-cabinet.md` is still owed); the decision record for the public history predating [H3084](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H3084-Opus_Systema_teacher-payout-truth-recording-course_18.08.26.md) (Opus 5, Systema teacher payout truth recording course) is [DECISIONS_systema-public-history-staff-data.md](https://github.com/gasyoun/Uprava/blob/main/docs/DECISIONS_systema-public-history-staff-data.md).

4. **Secrets + history.** Quick scan of tree AND history for keys/tokens (`git log -S` spot checks; `gh api repos/gasyoun/<repo>/secret-scanning/alerts` — check merge refs too, not just branches). A hit here escalates to [`/secret-purge`](https://github.com/gasyoun/claude-config/blob/main/commands/secret-purge.md), and publication waits.

5. **Gitignored-bulk leakage.** Data that is gitignored-by-design locally (regenerable bulk stores, personal fix-lists) must not ride along via the publish mechanism (Pages build outputs, release assets, `gh-pages` branches often bypass `.gitignore` reflexes).

## Verdict

- **GO** — state what was checked (including rights facts recorded or already on file) and proceed with the publish action. **Rights uncertainty alone is GO.**
- **NO-GO** — only when one of these is present: **confirmed prohibition**, **explicit restricted-data designation**, **privacy exposure**, **platform-policy / secrets / intentional private-only visibility / bulk-leakage**. Name the blocking item(s), do NOT publish. Add a GTD row only for a *real* human action (e.g. rotate a secret), **never** for “please adjudicate unclear rights.”

New standing visibility facts discovered during the check get appended to the intended-visibility list in this file and, if broader, to [Uprava/FINDINGS.md](https://github.com/gasyoun/Uprava/blob/main/FINDINGS.md) via [`/findings-append`](https://github.com/gasyoun/claude-config/blob/main/commands/findings-append.md). New **known** rights facts go into the asset's durable rights record once — not into a reopened adjudication loop.

_Dr. Mārcis Gasūns_
