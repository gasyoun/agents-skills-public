---
name: "acl-uplift"
description: 'Execute ONE ACL-grade methodology-uplift item (benchmark fits, field metrics, annotator-validity fixes, companions). Judgment Opus 4.8 (claude-opus-4-8); reframes Fable 5 (claude-fable-5); metric builds Sonnet 5 (claude-sonnet-5).'

---

# acl-uplift

Use this skill when the user asks to run the migrated source command `acl-uplift`.

## Command Template


# /acl-uplift <memo item | ruling | paper Axx + uplift>

_Created: 17-07-2026 · Last updated: 25-08-2026_

The methodology-uplift execution discipline improvised across the SanskritSpellCheck ACL wave (H825 cross-family annotator + GED eval, H826 S-curve/DTE benchmark, H827 tied-field detector from a named ancestor paper, H828 Oflazer/Brill-Moore suggester spec) and its ancestors (H178 pwg_ru four-way eval bake-off, H281 csl-guides standards uplift). `/dh-memo` (or a roadmap interview) AUTHORS the agenda; this skill EXECUTES one item of it — the agenda item is the argument, and one session = one item, never the whole memo.

Argument: the memo item / interview ruling (file + item number, or a pasted ruling), or `<Axx> <uplift description>`. Resolve the paper's row in [Uprava/ARTICLES.md](https://github.com/gasyoun/Uprava/blob/main/ARTICLES.md) and the owning repo before anything else.

Evidence base: H825 H826 H827 H828 (SanskritSpellCheck wave, 12-07-2026) + H178 H281 (historical). Dedupe: `/dh-memo` authors the agenda (memo-only, no build); `/paper-referee` reviews a finished draft hostilely; `/eval-harness` builds a generic eval loop — this skill owns the *methodology delta* between what the paper does and what the field's reviewers require, executed end-to-end.

## Phase 0 — Anchor + prior art

- Read the memo item / ruling verbatim; read the paper's ARTICLES.md row + the owning repo's roadmap/`.ai_state.md` WIP + GTD for a `🔒 CLAIMED` tag (concurrent-session check).
- **Reuse the existing ACL survey, never re-derive it** (H281 rule): the org has prior crosswalks (H173 placement pass, H265/H269 standards surveys, the repo's own /dh-memo). `/prior-art` + `hub_grep.py` before mining aclanthology.org fresh; extend, don't repeat.
- **Read the repo's refuted/locked registries first** (DEAD_ENDS, ROADMAP rulings): every H825–H828 handoff carried explicit "refuted R#" fences, and an uplift that quietly re-litigates a refuted design is a defect, not initiative. Quote the fence in your plan.
- Write the `/goal` stop condition before executing (every evidence session had one).

## Phase 1 — Classify the uplift shape

Name which of the four recurring shapes the item is (mixes happen; name the primary):

1. **Benchmark fit** — re-express existing results in a published shared task's reporting terms (SemEval-2015 DTE epoch/distance bands, SIGHAN CSC, QALB, WMT QE) so the system is placeable against known work. No re-running the system; it is a *reporting* transform plus the citation.
2. **Field-standard metrics** — extend the committed eval script to emit what reviewers expect: detection-level P/R/**F0.5** + FPR for detection tasks, word-accuracy/CER for normalization, κ/Krippendorff's α for agreement, reference-free QE (COMETKiwi) where no reference exists — plus, where damage matters, a **harm metric** (fraction of already-correct tokens wrongly changed).
3. **Annotator validity** — fix self-preference confounds: add a **cross-family** second annotator (non-Anthropic; DeepSeek per the standing key rule — never re-ask for an Anthropic key), **blind** it to pass provenance, anchor to a small human-labelled seed, and report agreement with **honest channel labels** (human×model / model×model — never relabeled human×human).
4. **Reframe / companion artifact** — reposition the paper on the canonical task spine the literature already owns (GED/confusion-set, historical text normalization, noisy-channel correction), with 2–4 verified citations per claim; or author the companion the venue expects — workshop short paper (LChange, LaTeCH-CLfL), data statement (Bender & Friedman), reproducibility checklist, error taxonomy recast as CL error analysis, or a spec-only design doc when the build is deferred by ruling.

## Phase 2 — Execute the one item

- **Metrics/benchmark builds:** extend the repo's *committed* eval script (never a throwaway shell run) so every reported number is recomputable; persist the results table to the repo's findings/results doc in the same pass.
- **The two-gates principle (H825):** a field-standard research metric and the repo's shipping/filing gate are *different roles, not a conflict* — report F0.5/FPR for the paper AND keep the existing hard gate (e.g. FP=0 against the nochange whitelist) green. Never relax a shipping gate to make a research metric look conventional.
- **Reframes:** rewrite the framing section on the canonical spine; add the named ancestor papers (the direct-ancestor hunt is half the value — H827's Bloodgood & Strauss find produced a whole new detector); fix any live references defects found on the way (missing keystone, fabricated citation).
- **Citations are live-verified, every one clickable** — fetch the anthology/arXiv page and confirm authors+title+venue before citing; a fabricated citation in a related-work section is the exact defect class this skill exists to remove (H825 found one in-flight).
- **Spec-only items stay spec-only:** when the ruling defers the build, the deliverable is a committed spec a future session can execute from — architecture, reuse map (what exists in-repo vs net-new), licensing guards, prior-art credit — and NO code (H828).

## Phase 3 — Verify

- Every new number: recomputable from a committed script, denominator stated (the H800 denominator-artifact lesson), model tier + exact version stamped per generated/judged batch.
- Every citation resolves; every agreement figure names its channels honestly; every existing hard gate re-run and green.
- Unvalidated-instrument caveats stated in the doc itself (e.g. COMETKiwi is unvalidated on 19th-c. dictionary glosses — a triangulation aid, not a headline metric).

## Phase 4 — Deliver

- Branch + PR + auto-merge (worktree if the repo is contention-guarded); results tables persisted; `/articles-update` the paper's row; changelog entry (+ `/cut-release` where the repo cuts releases); GTD mirror for any @DECIDE (venue choice, DOI mint) the uplift surfaces; close the handoff via [`/handoff-close --reconcile`](https://github.com/gasyoun/claude-config/blob/main/commands/handoff-close.md) (H1342 — the row flip + close clause buffer into the session's single `registry_reconcile.py flush`); `/findings-append` any durable non-obvious find (a new ancestor paper, a metric gotcha).

_Dr. Mārcis Gasūns_

_Dr. Mārcis Gasūns_
