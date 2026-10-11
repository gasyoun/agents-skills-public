---
name: "panini-sutra-lookup"
description: 'Resolve an Aṣṭādhyāyī sūtra ref (a.p.n) to Devanagari, IAST, S.C. Vasu gloss, sūtrārtha, optional Kāśikā/Mahābhāṣya, ready-to-paste citation + ashtadhyayi.com URL. Never recite sūtra numbers — always resolve.'

---

# panini-sutra-lookup

Use this skill when the user asks to run the migrated source command `panini-sutra-lookup`.

## Command Template


# /panini-sutra-lookup

_Created: 09-07-2026 · Last updated: 26-08-2026_

Grammar-rule claims in Sanskrit prose (e.g. the GasunsDhatu dissertation) need an
exact Aṣṭādhyāyī reference — `ср. Aṣṭādhyāyī 6.1.68` — which must **never** be recited
from memory (sūtra numbering is unforgiving). This skill resolves references and keywords
against the canonical public dataset that powers ashtadhyayi.com, so the citation is real.

**Prior art / source of truth:** [ashtadhyayi-com/data](https://github.com/ashtadhyayi-com/data)
`sutraani/*.txt` (JSON) — no scraping. Resolver:
[panini_sutra.py](https://github.com/gasyoun/SanskritGrammar/blob/main/GasunsDhatu_2014/revision-2026/panini_sutra.py).
Do not reinvent a sūtra list; consume this one.

## Argument

- A reference `a.p.n` (adhyāya.pāda.sūtra), e.g. `6.1.68`, `1.3.1`.
- Or `--search <term>` where the term is Devanagari (`वृद्धि`), IAST/roman (`vruddhi`), or a
  fragment of the romanized sūtra.

## How to run

From the repo (any OS; the script self-caches the dataset under `revision-2026/panini_cache/`):

```
python GasunsDhatu_2014/revision-2026/panini_sutra.py 6.1.68
python GasunsDhatu_2014/revision-2026/panini_sutra.py 1.3.1 --commentary
python GasunsDhatu_2014/revision-2026/panini_sutra.py --search "vruddhi" --limit 5
python GasunsDhatu_2014/revision-2026/panini_sutra.py 6.1.68 --refresh   # re-pull cache
```

Flags: `--commentary` adds Kāśikā + Mahābhāṣya (bhashya) + Kātyāyana-vārttika text mapped to
that sūtra; `--limit N` caps keyword hits; `--refresh` re-downloads the cache.

## Reference identity (so you can sanity-check a hit)

- id `i` is 5-digit `a·p·nnn`: `6.1.68` → `61068`, `1.3.1` → `13001`.
- citation string: `ср. Aṣṭādhyāyī {a}.{p}.{n}`.
- site URL: `https://ashtadhyayi.com/sutraani/{a}/{p}/{n}` — a real, openable link, so render it
  as a link, not inert text.

## Workflow (RWS panini-traditional edits)

1. For each `missing_sutra_reference` place in the text, identify the rule being asserted.
2. `--search` the rule's keyword (or, if the sūtra is known, resolve the reference directly);
   confirm the Devanagari/gloss matches the claim before citing — a plausible-looking number
   is not enough.
3. Paste the returned `ср. Aṣṭādhyāyī a.p.n` (optionally + Cardona/other secondary ref) at the
   claim. Log the change through the repo's errata convention (do not hand-edit the print `.mdx`
   mid-revision — see [errata.yml](https://github.com/gasyoun/SanskritGrammar/blob/main/GasunsDhatu_2014/errata.yml)
   and RWS_REPORT.md §6).

## Scope / caveats

- Resolves and cites; it does **not** interpret whether the cited sūtra actually licenses the
  author's claim — that is a human/scholarly judgement. Confirm the gloss fits before pasting.
- The bundled `--commentary` Mahābhāṣya/Vārttika text is the ashtadhyayi.com edition, keyed
  per-sūtra; for the deeper commentary-corpus need (Kielhorn Mahābhāṣya, Pataskar vārttika,
  Cardona/Staal/Scharfe secondary literature) use [`/panini-commentary-corpus`](https://github.com/gasyoun/claude-config/blob/main/commands/panini-commentary-corpus.md)
  (shipped 09-07-2026, RWS_REPORT.md §6.7).
- Devanagari I/O needs a UTF-8 console; the script forces UTF-8 stdout/stderr.

_Skill authored by Opus 4.8 (`claude-opus-4-8`), 09-07-2026, H413._

_Dr. Mārcis Gasūns_
