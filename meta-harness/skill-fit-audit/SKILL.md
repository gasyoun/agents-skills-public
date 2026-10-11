---
name: "skill-fit-audit"
description: 'Score claudeskills.info + installed skills against hubs-derived org work profile, YES/MAYBE/NO manifest rows; --tool rules one tool vs house stack; human-gated per batch. Triggers: "skill-fit-audit", "X vs our own stack". Do not use to mine our own session history for gaps — that is /skill-mine; this scores third-party/installed skills against our work profile.'

---

# skill-fit-audit

Use this skill when the user asks to run the migrated source command `skill-fit-audit`.

## Command Template


# /skill-fit-audit [--category <name>] [--type skill|subagent|plugin|command|all] [--min-stars N] [--calibrate N] [--resume] [--tool <name>]

_Created: 15-07-2026 · Last updated: 05-09-2026_

Answers one question at scale: **of the thousands of published Claude skills, which ones
actually fit the work done across this org — and which don't?** It compares two catalogues at
once — the public [claudeskills.info](https://claudeskills.info) marketplace (≈15k items, 33
categories) for *discovery*, and your own installed skill set for *redundancy* — against a work
profile derived from the org's own hubs. Every evaluated skill lands in a durable, ranked
YES / MAYBE / NO table with a reason and an install link, so "the yes and the no" are documented,
not just discussed.

The unit of work is **one category per run** (there are 33). A full marketplace sweep can't and
shouldn't happen in one sitting, so the skill is built around a **resumable manifest**: pick a
category, get its ranked verdicts appended, review, then pick the next. `--resume` continues where
you left off.

**Arguments** — `--category` (run this one marketplace category; omit to be shown the ranked
category menu first; **skills only — subagents/plugins/commands are uncategorized, see below**);
`--type` (limit to `skill`/`subagent`/`plugin`/`command`, default `all`); `--min-stars` (drop
long-tail repos below a star floor to cut noise, default `0`); `--calibrate N` (size of the
random discard sample in Phase 3, default `20`; `--calibrate 0` skips it and forces the manifest
to record `discard rate: UNMEASURED`); `--resume` (continue the existing manifest instead of
re-ranking categories).

## The API (claudeskills.info)

Real, unauthenticated JSON endpoints — call them with WebFetch:

- **`GET /api/v1/meta`** → `{ total_items, unique_repos, categories[], types{}, sorts[], endpoints{} }`.
  Use once per run to get the live `categories` list and counts (do NOT hardcode the 33 names —
  they drift).
- **`GET /api/v1/search?q=&category=&type=&sort=&limit=&offset=`** →
  `{ total, limit, offset, results[] }`. Page a whole category with `category=<name>&limit=50` and
  increasing `offset` until `offset ≥ total`. Each result object:

  | field | use |
  |---|---|
  | `slug` / `name` | identity + dedupe key |
  | `description` | the trigger/purpose text you score against the profile |
  | `category` / `type` | which bucket + skill vs subagent vs plugin vs command |
  | `confidence` | catalogue's own quality signal (`high`/…) — a tiebreaker, not a verdict |
  | `stars` | repo popularity — a maintenance/adoption proxy, apply `--min-stars` here |
  | `source.repo` / `source.url` | the install link that goes in the table |

WebFetch caches 15 min per URL and answers via a small model, so ask it to **return the raw
`results` JSON verbatim** (not a summary) — you need the fields intact to score them. For a
whole-type sweep (thousands of rows) WebFetch is the wrong tool: page the API from a small
Python script instead and keep the JSON on disk, so the scoring agents read a local shard
rather than re-fetching.

### ⚠️ `category` is a SKILLS-ONLY facet — subagents are uncategorized

Measured 17-07-2026 ([H1161](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1161-Opus_Uprava_subagent-fit-audit-claudeskills_17.07.26.md) (Opus 4.8 (`claude-opus-4-8`)) — subagent-fit audit of claudeskills.info) and load-bearing for `--type subagent`:

| Query | `total` |
|---|---|
| `?type=subagent` | 2179 |
| `?type=subagent&category=research` (and `documentation`, `workflow`, `testing`, `meta`) | **0** |
| `?type=skill&category=research` / `documentation` | 50 / 71 |

**0 of 2,179 subagent records carry a non-empty `category`.** So `--category <x> --type subagent`
matches nothing, and the category-per-run design below **does not apply to subagents** — Phase 1
is skippable for them. Batch a non-skill type by **`offset` sharding** (`?type=subagent&limit=100`,
walk `offset` to `total`), then classify into your own slices from the descriptions. Do not
hardcode this table either: re-measure per run, since the site may add categories later.

## Phase 0 — Build the work profile + the redundancy set (once per manifest)

Two reads, both cached in the manifest header so later categories reuse them:

1. **Work profile** — what this org actually does, from its own hubs (full blob URLs; read what's
   present, skip what isn't):
   - [github-spine/CLAUDE.md](https://github.com/gasyoun/github-spine/blob/main/CLAUDE.md) and the org [CLAUDE.md](https://github.com/gasyoun/github-spine/blob/main/CLAUDE.md) cross-repo spine
   - [Uprava/MEGABOOK.md](https://github.com/gasyoun/Uprava/blob/main/MEGABOOK.md) — the 21 theses / sources of truth
   - [Uprava/PROJECT_INTERLINKS.md](https://github.com/gasyoun/Uprava/blob/main/PROJECT_INTERLINKS.md) — who owns what feed
   - [SanskritLexicography/FEATURES_INDEX.md](https://github.com/gasyoun/SanskritLexicography/blob/master/FEATURES_INDEX.md) — 44 dicts · 20 interfaces · 37 datasets · 14 tools that EXIST
   - [github-spine/SHARED_CODE.md](https://github.com/gasyoun/github-spine/blob/main/SHARED_CODE.md) — canonical helper families

   Distil a **profile fingerprint**: recurring task types (OCR/masking, XML correction, alignment,
   morphology, Devanāgarī transcoding, lexicon markup, dictionary triage, docusaurus publishing,
   citation/BibTeX, review-sheet gating, cross-repo hubs, PDF/scan work, Russian↔Sanskrit
   translation), languages (Python on Windows, PHP, mdx), and platforms (Cologne/CSL, docusaurus,
   Cloudflare). Write it into the manifest header as a bullet list — this is the yardstick every
   skill is scored against.

2. **Redundancy set** — what you ALREADY have, so the audit never says "adopt X" when `/x` exists:
   - [github-spine/SKILLS_INDEX.md](https://github.com/gasyoun/github-spine/blob/main/SKILLS_INDEX.md) — the 120-skill catalogue
   - the harness available-skills list this session
   - `~/.claude/commands/*.md` and `~/.claude/skills/*/SKILL.md` on disk (Glob)

   Keep a set of `{name → one-line purpose}`. Any marketplace skill whose purpose is already met
   here is a **NO — redundant**, naming the covering skill.

## Phase 1 — Rank the categories, let a human pick (skip with `--category`)

`GET /api/v1/meta`, take the live `categories[]` with their counts, and **rank them by relevance
to the profile fingerprint** (Security, Documentation, Data Analysis, DevOps, Testing, Research,
Writing tend high for this org; Gaming, Finance, Web3, Email tend low). Present the ranked list —
category · item count · why it's likely relevant — and ask which one to run this pass (default:
the top-ranked not-yet-done category). One category per run is the whole point; don't fan out
across all 33 unprompted.

## Phase 2 — Score the chosen category → append the ranked table (the batch)

Page the category (`category=<name>&limit=50`, walk `offset`), apply `--type`/`--min-stars`
filters, dedupe by `slug`, and score each surviving item against the profile + redundancy set with
this rubric:

| Verdict | Meaning | Fit |
|---|---|---|
| **YES** | Hits a *recurring* profile task, NOT already covered by an installed skill, with a concrete adoption path | ≥7/10 |
| **MAYBE** | Plausibly useful but niche/occasional, OR partially overlaps an existing skill, OR quality/maintenance unclear (low stars, `confidence` not high) | 4–6/10 |
| **NO** | Irrelevant to the profile, OR fully redundant with an installed skill (name it), OR abandoned/thin | ≤3/10 |

Append to the manifest, **sorted YES → MAYBE → NO**, one row per skill:

```
| Skill | Type | ⭐ | Verdict | Fit | Why (one line) | Overlaps | Source |
```

- `Overlaps` = the installed `/skill` it duplicates, or "—".
- `Source` = a full `source.url` blob/repo link (clickable, per repo convention).
- Every evaluated skill gets a row — the NOs are the point as much as the YESes ("document all
  the no and yes").

Never cap the survivor set at a round number to "bound cost". In H1161 an arbitrary cap of 40
MAYBEs silently cut the single most org-relevant cluster in the batch (three OCR-against-scan-image
agents and the one Docusaurus agent). If you must cap, `log()` exactly what was dropped and carry
it into the manifest as **unverified**, never as NO.

## Phase 2b — Body-read every survivor (descriptions are NOT evidence)

**Every YES/MAYBE gets its real source fetched and read before it reaches the manifest**: take
`source.url`, fetch the raw file (`raw.githubusercontent.com`, or swap `/blob/` for `/raw/`), and
judge the **actual frontmatter + body prompt**. Verdicts: **ADOPT** (vendor + localize) ·
**INSPIRE** (implementation wrong for us, shape worth rebuilding org-native — say which shape) ·
**DROP**. Default to DROP when uncertain.

This phase is not optional, because catalogue descriptions are unreliable **in both directions**
(all measured in H1161):

- **Over-sell** — `doublecheck` is "a description-level illusion of structure"; `agent-evaluator`
  delegates its whole rubric to files that don't exist outside its home repo.
- **Mis-describe** — `technical-content-evaluator`'s description sells a "curriculum architect";
  its body is a repo-honesty auditor. Different agent entirely.
- **Under-sell** — `requirements-verifier` reads as an Elixir/Phoenix plugin sourcing requirements
  from Linear (any keyword rule ⇒ OFF), yet its body carries a genuinely useful spec→diff
  coverage protocol. It became one of the few INSPIREs.

The lesson generalises: **descriptions advertise domain, bodies carry protocol.** A rubric that
scores descriptions is scoring the wrong feature. Note also that H999's GO tier still had a ~25%
false-positive rate that only hands-on trial caught ([H999](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H999-Opus_Uprava_skill-fit-audit-claudeskills_16.07.26.md) (Opus 4.8 (`claude-opus-4-8`)) — skill-fit audit of claudeskills.info, GO-tier trial) — so a body-read survivor is `not yet trialled`
in the manifest, never `adopt`.

## Phase 3 — Calibrate the discards (MANDATORY; the audit is not done without it)

An audit that measures only its keeps has no error rate, and its headline ("we looked at N") is
really "we looked at the survivors and guessed about the rest". So **measure the discards**:

1. Take a **random sample of the NO/OFF bucket** — `--calibrate N`, default 20, seeded and recorded
   so the draw is reproducible and can't be cherry-picked.
2. Deep-read each one exactly as in Phase 2b, **independently** — the judge is told the item's
   description and body, NOT that it was already discarded, and NOT the discarding rationale.
3. Score each `flip` / `no-flip`. A **flip** = the real body reveals genuine relevance the
   description hid (the `requirements-verifier` failure mode: *body-relevant, description-misleading*).
4. **Publish the flip rate in the manifest, next to the headline.** It is the audit's error bar.

Read the result honestly, and state the bound rather than implying certainty:

- **0/20 flips** — the discard rule is defensible; report it as "≤14% at 95% confidence
  (rule of three)", not as "0%". A zero sample is a bound, not a proof.
- **≥2/20 flips** — the discard rule is broken. Say so, re-scan the bucket on *protocol* features
  (shape words: hypothesis, adjudicate, provenance, evidence tier, coverage, spec-vs-diff, gold,
  inter-annotator, alignment, corpus, XML/TEI, OCR, transliteration, philolog\*, lexicograph\*,
  morpholog\*) rather than domain words, and deep-read anything hitting ≥2.

A cheap free companion check: the protocol-feature re-scan costs **zero fetches** (it reads
descriptions already on disk) and directly targets the failure Phase 2b documents. Run it whenever
the discard bucket is large. In H1161 it returned 1 hit in 2,124 — which is itself the finding that
the corpus contains almost no philological vocabulary.

**If the calibration cannot run** (API failures, auth errors, budget), the manifest says
`discard rate: UNMEASURED` and the audit is explicitly **incomplete**. Never quietly ship a keep-only
result as if it were a full sweep.

Then **stop and summarise**: `<batch>: N scored → x YES, y MAYBE, z NO`, the YES names, and the
measured discard flip rate. Wait for the human to pick the next batch. Never auto-advance.

## The manifest (durable output)

One control doc, appended across runs:
[`Uprava/SKILL_FIT_AUDIT_CLAUDESKILLS.md`](https://github.com/gasyoun/Uprava/blob/main/SKILL_FIT_AUDIT_CLAUDESKILLS.md)
(private hub). Structure:

1. **Header** — dated line + `_Dr. Mārcis Gasūns_` byline, the model tier+version that ran it, the
   profile fingerprint (Phase 0.1), and the catalogue snapshot (`total_items`, date).
2. **Progress table** — one row per batch: name · item count · status (done/pending) · YES/MAYBE/NO
   counts · **discard flip rate (n/N)** · date run. This is what `--resume` reads to know what's
   left. A batch with no flip rate is `pending`, not `done` — an unmeasured discard bucket means
   the batch was never really finished.
3. **Per-category ranked tables** — the Phase-2 output, newest appended.
4. **Adopt shortlist** — a running promoted list of the YES verdicts with the concrete next step
   (which repo, `/prior-art` first, install command).

Because it's a measured result table across the org, it obeys the persist-tables + document-findings
reflex: it lives in the hub, not the chat. On any run that produces new YES verdicts, add an
`[Unreleased]` bullet to Uprava's CHANGELOG and update the row in
[Uprava/GTD_NEXT_ACTIONS.md](https://github.com/gasyoun/Uprava/blob/main/GTD_NEXT_ACTIONS.md) if a
YES becomes an action.

## Mode `--tool <name>` — one external tool, adopt or decline (U43, MG vote 12-08-2026)

**Skill-mine U43** ([census H2323–H2591](https://github.com/gasyoun/Uprava/blob/main/archive/SKILL_CANDIDATES_CENSUS_H2323_H2591_12_08_2026.md), [H2603](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H2603-Opus_claude-config_skill-update-u43-skill-fit-audit-single-tool_12.08.26.md) (Opus 4.8 (`claude-opus-4-8`)) — skill-update U43: single-tool mode for skill-fit-audit). Everything above batches the ≈15k-item marketplace. This mode rules on **one third-party tool or service against the house stack** — memora and claude-mem vs the house memory stack, an external RAG vs the hub-grep/recall path (H2419 · H2547 · H2447). Three sessions, no owning skill, and two of them re-argued the same memory question from scratch.

It is deliberately **not** a mini-batch. The batch mode asks "which of these many things fit?"; this asks "**should we replace something we already run?**" — a different and stricter question.

1. **Name the estate constraint the tool claims to remove**, in the org's own measured terms (tokens/session, recall latency, a failure the current stack demonstrably has). A tool evaluated against a constraint nobody measured is being evaluated against a wish.
2. **Describe what already covers it**, honestly, including the parts that cover it badly. The house stack's weaknesses belong in the comparison, or the ruling is rigged toward the incumbent.
3. **Read the source.** Same untrusted-code rule as the batch mode; no adopt on a README.
4. **Rule adopt / decline / adopt-in-part**, and write the reasoning down — the point of the mode is that the question stops being re-litigated. Two of the three evidence sessions were the same question asked twice.
5. **A decline is a durable artifact too.** Record what would change the answer (a version, a price, a measured constraint crossing a threshold), so a future session knows whether to revisit instead of re-deriving.
6. **Adopt-in-part is often the honest verdict**: take the idea, not the dependency. Name which house component gains the borrowed behaviour and mint the handoff for it.

Output: a dated ruling in the org's decision home ([`/decision-record`](https://github.com/gasyoun/claude-config/blob/main/commands/decision-record.md) where a formal ADR fits), plus the `@DECIDE`→resolved row if a human ruling gated it. Not a manifest row — this mode produces one ruling, not a batch table.

## Guardrails

- **Discovery ≠ endorsement.** A marketplace YES means "worth trying", never "install unreviewed".
  Third-party skills are untrusted code from arbitrary repos — the adopt step routes through
  `/prior-art` and a human read of the source before anything is wired in.
- **No hardcoded catalogue.** Category names, counts, and fields come from the live API each run;
  the numbers above are illustrative and drift.
- **Redundancy first.** Before ranking a batch, the installed-skills set must be loaded — the
  most valuable NO is "you already built this".
- **Measure the discards, not just the keeps.** Phase 3 is mandatory. Keep-only audits are how a
  0.18%-adopt headline gets mistaken for "we assessed 2,179 items" when it really means "we
  assessed 55 and guessed about 2,124".
- **Beware the rubric closing the door.** A profile + a ~120-skill redundancy set together define
  the admissible set as *"org work the org has never automated"* — near-empty **by construction**.
  If nearly every DROP terminates in "redundant" rather than "thin/off-profile", the rubric is
  producing the verdict, not the evidence. Report those two legs separately.
- **DROP means "don't install", not "nothing here".** An item can be correctly DROPped and still
  name a live uncovered gap in its `gap` field — that gap belongs in the org-native backlog, not
  in the bin with the item. Harvest gaps from DROPs before closing a batch.
- **Expect zero ORCH adoptions, permanently.** Orchestration is what the main loop *is*; an agent
  that routes/grades/supervises other agents cannot be a read-only fan-out worker. ORCH "gaps"
  re-shape into a skill or a script, never a subagent (H1161: 190 ORCH scanned, 0 adopted).

_Dr. Mārcis Gasūns_
