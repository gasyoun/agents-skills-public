---
name: "sandhi-gold-audit"
description: 'Audit a hand-annotated sandhi rule table (gita_sandhi.tsv) against corpus-induced rules — malformed entries, corpus-unattested rules (likely typos), coverage gaps. Triggers: "sandhi-gold-audit", "audit gita_sandhi.tsv". Do not use on free-form authored text — this audits a specific hand-annotated gold TSV against corpus rules; use /sandhi-check for prose.'

---

# sandhi-gold-audit

Use this skill when the user asks to run the migrated source command `sandhi-gold-audit`.

## Command Template


# /sandhi-gold-audit

_Created: 14-07-2026 · Last updated: 05-09-2026_

## Default path (seconds — run this first)

```sh
python ~/Documents/GitHub/Uprava/tools/sandhi_gold_audit.py [hand.tsv] [--corpus path] [--min-count 50]
python ~/Documents/GitHub/Uprava/tools/sandhi_gold_audit.py --selftest
```

Script: [`Uprava/tools/sandhi_gold_audit.py`](https://github.com/gasyoun/Uprava/blob/main/tools/sandhi_gold_audit.py) (ported in [H1967](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H1967-Grok_Uprava_m2s-wave2-thin-gate-ports_30.07.26.md) — Grok 4.5, wave-2 thin-gate script ports).

Defaults: hand = `kosha/data/gita/gita_sandhi.tsv`, corpus = `kosha/data/sandhi/corpus_sandhi.tsv`.

## Report classes (script)

| Class | Meaning | Action |
|---|---|---|
| **malformed** | well-formedness fail (Cyrillic, missing spaces, empty side) | fix in gold via PR |
| **unattested** | hand rule never in corpus | verify — not auto-wrong (rare/Vedic OK) |
| **missing** | high-count corpus rule absent from hand | candidate add |

## Judgment residue

"Not in the corpus" ≠ "wrong". Separate malformed (fix) from unattested (verify) from rare-but-real (keep). Apply gold fixes via PR only when asked; the script never rewrites the table.

_Dr. Mārcis Gasūns_
