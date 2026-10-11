---
name: "panini-commentary-corpus"
description: 'Thematic search of Paninian commentaries (Mahabhashya, varttikas, Kashika) by ROOT/CONCEPT: commentator frames, school attribution, Cardona/Staal pointers; above /panini-sutra-lookup. RWS lidova edits; Sundara crux lookups.'

---

# panini-commentary-corpus

Use this skill when the user asks to run the migrated source command `panini-commentary-corpus`.

## Command Template


# /panini-commentary-corpus

_Created: 09-07-2026 · Last updated: 26-08-2026_

A `lidova-commentary`-style claim in the dissertation ("in which school did this
notion of the root form, how did the bhāṣya/vārttikas develop or contest views on
roots, which authorities stood behind the classifications") is not resolvable from
memory and not resolvable by sūtra-number alone. This skill searches the commentary
corpus **thematically** — by a root or a concept — and lays out the commentator
frame with a real secondary-literature pointer.

**Prior art / source of truth (do not rebuild):**
- Core commentary data: [ashtadhyayi-com/data](https://github.com/ashtadhyayi-com/data)
  `sutraani/{bhashya,vartika,kashika}.txt` (JSON) — the same dataset cached by
  [panini_sutra.py](https://github.com/gasyoun/SanskritGrammar/blob/main/GasunsDhatu_2014/revision-2026/panini_sutra.py)
  (H413 — [Opus](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H413-Opus_SanskritGrammar_gasuns-dhatu-rws-visa_09.07.26.md), GasunsDhatu RWS visa). No scraping.
- This tool is the layer **above** `/panini-sutra-lookup`: resolver
  [commentary_corpus.py](https://github.com/gasyoun/SanskritGrammar/blob/main/GasunsDhatu_2014/revision-2026/commentary_corpus.py),
  reusing panini_sutra.py's fetch/cache/util helpers + vidyut for transliteration.
- Secondary-literature pointer index (curated, pointers only — rights):
  [secondary_lit.json](https://github.com/gasyoun/SanskritGrammar/blob/main/GasunsDhatu_2014/revision-2026/secondary_lit.json).

## Argument

A **root or concept** term — Devanagari (`धातु`), IAST (`dhātu`), Harvard-Kyoto
(`dhAtu`), or SLP1. The scheme is auto-detected (vidyut) and transliterated to
Devanagari for matching against the commentary text.

## How to run

From the repo (any OS; the script self-caches the dataset under
`revision-2026/panini_cache/`):

```
python GasunsDhatu_2014/revision-2026/commentary_corpus.py --search "dhātu" --limit 6
python GasunsDhatu_2014/revision-2026/commentary_corpus.py --search "आत्मनेपद" --layer vartika
python GasunsDhatu_2014/revision-2026/commentary_corpus.py --frame "vṛddhi"
python GasunsDhatu_2014/revision-2026/commentary_corpus.py --biblio           # whole index
python GasunsDhatu_2014/revision-2026/commentary_corpus.py --biblio vartika   # one topic
```

- `--search TERM` — which sūtras' commentary (or sūtra text) contains TERM, ranked
  by how many layers matched, each hit showing the sūtra + the matching snippet per
  layer + the `ср. Aṣṭādhyāyī a.p.n` citation and ashtadhyayi.com URL.
- `--frame TERM` — the search hits **plus** a synthesized commentator frame: the
  school, the sūtra→vārttika→bhāṣya→Kāśikā chain, the "who developed/contested"
  reading, the Cardona/Staal/Scharfe pointers for the topic, and a ready-to-adapt
  dissertation footnote.
- `--biblio [TOPIC]` — the secondary-literature pointer index alone.
- `--layer all|bhashya,vartika,kashika` — restrict `--search` to layers.
- `--limit N` — cap the number of sūtras shown.

## Reference identity (sanity-check a hit)

- id `i` is 5-digit `a·p·nnn`: `1.3.12` → `13012`. The vārttika layer is keyed by
  the `a.p.n` reference string upstream and re-indexed to the 5-digit id here.
- citation string: `ср. Aṣṭādhyāyī {a}.{p}.{n}`.
- site URL: `https://ashtadhyayi.com/sutraani/{a}/{p}/{n}` — a real link; render it
  as a link, not inert text.

## Workflow (RWS `lidova-commentary` edits)

1. Identify the root/concept the passage is about (e.g. the notion of *dhātu*, the
   seṭ/aniṭ split, ātmanepada assignment).
2. `--frame "<term>"` to get the sūtra chain + which vārttika/bhāṣya contest or
   settle the view + the topic's Cardona/Staal/Scharfe pointer.
3. Confirm the Devanagari/gloss actually fits the claim (surface substring matching
   can over-generate — see caveats), then compose the footnote from the emitted
   template, **verifying any page range against the physical copy** before it lands.
4. Log the edit through the repo's errata convention — do **not** hand-edit the print
   `.mdx` mid-revision (see
   [errata.yml](https://github.com/gasyoun/SanskritGrammar/blob/main/GasunsDhatu_2014/errata.yml)
   and [RWS_REPORT.md](https://github.com/gasyoun/SanskritGrammar/blob/main/GasunsDhatu_2014/revision-2026/RWS_REPORT.md) §6).

## Secondary use — episodic «Литпамятники»/Sundara lookups

Beyond the primary RWS `lidova-commentary` use, this skill serves **episodic** commentary-note
lookups for the Leonov Sundarakāṇḍa apparatus ([`/commentary-apparatus-ingest`](https://github.com/gasyoun/claude-config/blob/main/commands/commentary-apparatus-ingest.md),
[`/sundara-lexical-layer-qa`](https://github.com/gasyoun/claude-config/blob/main/commands/sundara-lexical-layer-qa.md)) — the rare epic verse whose note turns
on a Pāṇinian point (a parse the ṭīkās dispute in sūtra terms). Sundara is a «Литературные
памятники» edition, **not** a grammar-teaching edition, so this is occasional support, not a
standing layer; the RWS framing stays primary.

## Scope / caveats

- **Surface substring search**, not lemmatized: a *concept* term (vṛddhi, guṇa,
  ātmanepada, dhātu) matches its Devanagari string well; an inflected *root* form
  (bhavati, babhūva) will be missed by searching the citation form (भू). Read the
  snippet — a hit is a candidate, not a proof that the sūtra licenses the claim.
- The secondary-literature layer is **pointers, not full text** (rights). Named
  section/topic pointers are safe to cite; every emitted pointer carries an
  implicit "verify page range against the physical copy" — do not paste a page
  number this tool did not give you.
- It **cites and frames**; it does not adjudicate whether the tradition's siddhānta
  actually supports the author's reading — that is a scholarly judgement.
- The Pataskar vārttika PDF and the Kielhorn/Cardona Mahābhāṣya TXT mirror named in
  the handoff are deeper corpora for manual consultation; the machine layer here is
  the ashtadhyayi.com per-sūtra edition, which is enough to locate the debate.
- Devanagari I/O needs a UTF-8 console; the script forces UTF-8 stdout/stderr.

_Skill authored by Opus 4.8 (`claude-opus-4-8`), 09-07-2026, H414._

_Dr. Mārcis Gasūns_
