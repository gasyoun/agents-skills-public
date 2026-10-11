---
name: "entry-anatomy"
description: 'Colour-coded anatomy of a Cologne (CDSL) headword: lemma/grammar/etymology/senses/citations into theme-aware HTML+JSON teaching assets; MW, PWG, AP90, GRA built-in. Triggers: "anatomy of an entry", "how to read a PWG/Apte entry".'

---

# entry-anatomy

Use this skill when the user asks to run the migrated source command `entry-anatomy`.

## Command Template


# /entry-anatomy

_Created: 12-07-2026 · Last updated: 29-08-2026_

Produce the **anatomy of a dictionary word entry** — the same labelled-dissection
deliverable Fable 5 (claude-fable-5) made for Duden, but for the Cologne Sanskrit dictionaries. Because
the CDSL source is *already structured markup* (not flat print), the parts are read
straight off the tags rather than guessed — the skill maps each markup class to a
tinted, legend-labelled region.

**Argument:** `<dict> <headword>` — e.g. `mw kAla`, `pwg agninakzatra`, `ap90 guru`,
`gra acitti`. The headword is the **SLP1 `<k1>` key** (the accent-stripped search form:
`kAla`, `guru`, `agni`), not Devanāgarī.

## What it produces

- A self-contained, theme-aware **HTML** page (light+dark) — one card per matching
  entry, every structural part colour-coded, with a legend explaining each part. Safe
  to embed in Docusaurus MDX (csl-guides), a Systema lesson, or kosha.
- A **JSON** token stream (`--format json`) for programmatic consumers — each token is
  `{part, text}`, so a site can re-render in its own component instead of using the
  bundled HTML.

## The parts it labels

headword/lemma (Devanāgarī + IAST + source key) · homonym index · grammar tag
(gender/POS) · etymology (root/derivation/compound analysis) · Sanskrit forms · sense
markers · gloss/definition · source citations (`RV.`, `MBh.`, `VS.` …) ·
cross-references (`cf.`/`Vgl.`/`q.v.`) · attested inflected forms with corpus loci
(Grassmann's signature).

## How to run

```
python ~/.claude/skills/entry-anatomy/entry_anatomy.py <dict> <k1> \
    [--csl-root PATH] [--format html|json|both] [--out FILE] \
    [--all-homonyms] [--title TEXT]
```

- `--csl-root` auto-detects a sibling `csl-orig/` checkout; pass it explicitly if the
  cwd is elsewhere.
- `--all-homonyms` renders every `<k1>` match (e.g. both `kāla`s), not just the first.
- Needs `indic_transliteration` for the Devanāgarī/IAST header; degrades to the raw
  `<k2>` key if the lib is absent.

## Covered dictionaries — and adding another

`mw` · `pwg` · `ap90` · `gra` ship with tag-maps in `DICT_MAPS`. The `<L>…<LEND>`
skeleton (`<k1>`/`<k2>`/`<pc>`, `¦` body separator, `<lex>`, `<ls>`, `<ab>`) is uniform
across all ~44 CDSL dicts; only the inline content markers differ (MW `<s>`/`<info>`;
PWG/AP90/GRA `{#…#}` Sanskrit + `{%…%}` gloss; AP90 `{@N@}` senses; GRA `<div n=…>`
attested forms). **To cover a new dict:** add one `DICT_MAPS[key]` entry with its
`name`, `inline` marker list, and `etym` regex — the render is shared, no other change.

## Render homes (consumers)

The skill is the **producer**; these are its intended consumers (each is a separate
wiring task, not done by the skill itself):
- **csl-guides** — a "How to read a Sanskrit dictionary entry" MDX page (public, SEO).
- **Systema-Sanscriticum** — an interactive learner-track lesson asset.
- **kosha** — entry-structure explainer alongside the API.
- **pwg_ru / pwg_en** (RussianTranslation) — anatomy of the German source entry beside
  its translation, as a translator aid.

## Caveats (state them, don't hide them)

- **Value is pedagogical/marketing**, not research — a teaching/onboarding/landing
  asset (Tier 0/1), not a data deliverable.
- **Nested markup** (a `<ls>` citation inside a `<lex>` grammar tag) is flattened into
  its outer part; the citation-ness of the inner span is not separately tinted. Fine
  for a teaching diagram, imperfect as a parser.
- **Etymology tagging is heuristic** (first etymology-shaped span); verify on a new
  dict before trusting it.
- **Devanāgarī rendering** relies on a system/site font (`Noto Sans Devanāgarī` /
  `Nirmala UI`); when embedding in an Artifact, inline the font as a data-URI (external
  fonts are CSP-blocked there).

**Visual counterpart:** for the Duden-style *annotated picture* — callout labels wired by
leader lines to an entry, print PDF + interactive web — use [`/entry-specimen`](https://github.com/gasyoun/claude-config/blob/main/commands/entry-specimen.md)
(this skill is the tinted-text breakdown; that one is the leader-line specimen sheet).

House provenance: built under
[H791](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H791-Opus_claude-config_entry-anatomy-dict-skill_12.07.26.md).

_Dr. Mārcis Gasūns_
