Origin: the key1-repair retraction (56/56 cards machine-resolvable — a/A, d/D
flattening; PWG itself printing «fehlerhaft für vidu» — while ~20 sheets of genuinely
hard questions sat unvoted). Org record:
[Uprava FINDINGS §442](https://github.com/gasyoun/Uprava/blob/main/FINDINGS.md).
`ambiguous` flag from a verdict. Specimen, in one session: the H2839 (Opus 5) — derived non-transcript asset from ustny korpus — path-name gate shipped
first as a 41-row TSV, then as a 24-row clustered TSV, and a human still could not vote it
(«Filling of TSV is not very easy… I still do not understand how to fill the TSV, with what
values»). The same batch as an HTML sheet was voted in ten minutes. If a batch deserves
human judgment it deserves the emitter — `csl_pyutil.render_review_sheet`, always.
What *does* change is **where the sheet lives, never whether it exists**:

| | PD sheet | ordinary sheet |
|---|---|---|
| Generate the sheet | **always** | always |
| Written to | a path **outside** the repo (refuse in-repo, exit 3) | repo/`review/` per the naming rule |
| Opened via | `file://`, offline, zero external requests | same, or the hub |
| `gasyoun.github.io/vote` | **not while unvoted third-party PD is on the cards** | default (H2774 (Opus 5) — vote hub h215-320 + xref publish, PWG first) |
| Committed | never (the sheet *is* the data) | per repo convention |
**Hosting after the vote is a different question, and usually a trivial one.** Once the
cards are ruled, the ones carrying no third-party data (a lecturer already public, an
author, a deity, a mis-parse) can be hosted like any other sheet. Publishing *unvoted*
third-party names to a public, crawlable, cached page is the one thing this rule does not
license — a human should decide that, and an agent should not carry it out on the org
default alone.
- ❌ «Вердикт скорера остается» — a position.
- ✅ «Вердикт скорера остается — статья продолжает репортить 0.96, расхождение остаётся
  видимым в §4.6 как иллюстрация; §5-оговорка о снисходительности разметчика усиливается» —
  a consequence.


**Show worked examples, plural.** One quoted item is not enough to judge a disputed rule or
label — quote several, or show the full set being ruled on. The vedic-r02 sheet quoted two
findings and MG's first note was «не хватает множества примеров».
**Only (d) cards are rendered.** The sheet must carry the (a)–(c) counts and a link to
their evidence file via the emitter's required `screening=` argument
([csl-pyutil ≥ 0.8.0](https://github.com/sanskrit-lexicon/csl-pyutil)):

```python
html = render_review_sheet(
    items=human_only_items,  # (d) only
    config={...},
    screening={
        "deterministic": 12,   # (a)
        "lookup": 4,           # (b)
        "agent": 30,           # (c)
        "human": len(human_only_items),  # (d)
        "evidence_path": "review/screening_evidence_<sheet_id>.md",
        "rules": ["dup-id-vs-h178_da", "gloss-slot-parity", "citation_tm"],
    },
)
```

Building with `extras=True` and no `screening=` **raises** — silence is not allowed. If you
truly screened nothing, pass zeros for (a)–(c), `human=len(items)`,
`rules=["none"]`, and an evidence path that says so (still a statement, not an omission).

**Worked examples (one per class):**

- **(a)** Card id already on the voted `h178_da` sheet → do not re-ask under mqm/likert/pairwise
  (A2 retired those arms). Evidence: byte-identical id set + prior `decisions.json`.
- **(b)** `<ls>R. 2,91,26</ls>` resolves in `SamudraManthanam/web/corpus.db` via
  `citation_tm.py` → put the attested SA + RU line on the card or auto-resolve the citation
  sub-question; do not make the human re-find what the TM already has.
- **(c)** ACC×NCC Tier C/D ([H1657](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1657-Opus_SanskritLexicography_acc-ncc-p2-agent-adjudication-49k_26.07.26.md) (Opus 5) — ACC×NCC P2 agent adjudication of 49k rows): agent adjudicates all rows with cited rules; human
  votes only the stratified blind spot-check (В2), never the full queue.
- **(d)** G6 gold starter / Renou precision gold / style-policy Q1–Q3 — the human label *is*
  the deliverable; no dataset substitutes for it.

