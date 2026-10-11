---
name: "klammerdiagramm"
description: 'Draw a publication-grade Klammerdiagramm - the German-Indological SVG bracket diagram of a long Sanskrit compound (word left-to-right, labelled binary nodes, printed-plate style); emits SVG + HTML with Charis. Triggers: "Klammerdiagramm", "bracket diagram".'

---

# klammerdiagramm

Use this skill when the user asks to run the migrated source command `klammerdiagramm`.

## Command Template


# /klammerdiagramm

_Created: 14-07-2026 · Last updated: 26-08-2026_

Produce the **Klammerdiagramm**: the classic German-Indological plate rendering of a long
Sanskrit compound — *long in members (lemmas combined), never in letters*: a two-lemma
title is a short compound however many letters it runs to — the word printed left→right,
hatched vertical brackets (two up-arrows each) under every member, and a descending
staircase of labelled binary nodes (A…) showing the right-branching nesting, with dashed
`I часть / II часть` spans on top. This is the
**publication SVG** deliverable — sibling of, not a mode of:

- [/sanskrit-parse](https://github.com/gasyoun/claude-config/blob/main/commands/sanskrit-parse.md) — ASCII staircase, comprehension parse;
- [/klammeruebersetzung](https://github.com/gasyoun/claude-config/blob/main/commands/klammeruebersetzung.md) — the woven bracket *translation*.

All three share ONE analysis method — the right-to-left (head-first) reading documented in
[sanskrit-right-to-left-reading.md (SamasaChakram)](https://github.com/gasyoun/SamasaChakram/blob/main/sanskrit-right-to-left-reading.md)
(§ *Publication-grade form — the Klammerdiagramm (SVG)* is this skill's rendering spec).
Never fork that analysis here. Illustrated gallery of every compound tested so far, with
configs and renders:
[klammerdiagramm-worked-examples.md (SamasaChakram)](https://github.com/gasyoun/SamasaChakram/blob/main/klammerdiagramm-worked-examples.md).

## Canonical tool

[klammerdiagramm/build_klammerdiagramm.py (SamasaChakram)](https://github.com/gasyoun/SamasaChakram/blob/main/klammerdiagramm/build_klammerdiagramm.py)
— config-driven, plate-fidelity geometry (H814 (Fable 5) — klammerdiagramm 1:1 plate fix, traced 1:1 from MG's printed plate):

```sh
# from the SamasaChakram clone root
python klammerdiagramm/build_klammerdiagramm.py                 # committed uddāma example (Leitan's constituency, D5)
python klammerdiagramm/build_klammerdiagramm.py --config c.json --out my-compound
python klammerdiagramm/build_klammerdiagramm.py --config klammerdiagramm/klammerdiagramm-uddama-plate-historical.json --out plate   # the superseded printed-plate reading
```

Emits `<out>.svg` **and** `<out>.html`. The HTML base64-embeds
[charis-klammer-subset.woff2](https://github.com/gasyoun/SamasaChakram/blob/main/klammerdiagramm/charis-klammer-subset.woff2)
(a WOFF2 subset of Charis 7.000, family renamed **"Klammer Serif"** because "Charis" is an
OFL Reserved Font Name; licence bundled as
[OFL-Charis.txt](https://github.com/gasyoun/SamasaChakram/blob/main/klammerdiagramm/OFL-Charis.txt),
provenance in
[make_charis_subset.py](https://github.com/gasyoun/SamasaChakram/blob/main/klammerdiagramm/make_charis_subset.py))
— so the HTML renders 1:1 on machines with **no** font installed. The bare SVG references
Charis by name and is for pipelines that already have the font. Theme-safe: `currentColor`,
`role="img"`, `<title>`/`<desc>`.

## Inputs to elicit — every run

1. **tokens** — the members in IAST, left→right (never re-transcode silently). Member
   positions are **auto-measured**: the generator sets the compound as one tightly
   hyphenated run (flush hyphens, exactly like the plate) and derives every x from real
   Charis advance widths read off the committed subset. Only pass explicit `cx` when you
   deliberately want the legacy spaced layout (members apart, hyphens floating).
2. **The binary nesting** (`nodes`: `[label, left, right, label_x]`, deepest first;
   children are token indices, or a node-label **string** naming an already-listed node).
   **Right-branching is the default, not a law** — deepest node = the rightmost pair, each
   leftward member accretes onto the accumulated right group. When the compound's analysis
   is ambiguous (a genuine left-nested sub-compound), ASK the user to confirm the
   bracketing before drawing — but never offer a left-to-right chain: **every samāsa binds
   right-to-left (MG ruling 15-07-2026); the dvandva alone is coordinate**, so a dvandva
   block is never chained — it renders FLAT: give the node a **list** of member indices —
   `["A", [0, 1, 2], null, 198]` — and all member bars drop into one shared band with a
   single label (worked `hasti-aśva-ratha-ghoṣaḥ` example). List members may themselves be
   node labels — a coordination of coordinations: sibling blocks share a rung via the
   optional 5th element (the depth level), and their bands join one rung lower —
   `["A", [0,1], null, 126, 0]`, `["B", [2,3], null, 345, 0]`,
   `["C", ["A","B"], null, 236, 1]` (worked `śīta-uṣṇa-sukha-duḥkha-dāḥ` example).
   **Sub-compounds may pre-form and join**: two token children plus an explicit level draw
   a standalone ∪ pair (`["A", 0, 1, 146, 0]` — e.g. a bahuvrīhi forming beside a
   karmadhāraya sibling), and a node whose children are BOTH node labels is the **directed
   join** — no lemma material of its own, thin Y-elbows to both children, head = the right
   child, read right-to-left (`["C", "A", "B", 258, 1]`; worked
   `cakra-pāṇi-caraṇa-kamala-sevā-phalam` example — a bahuvrīhi is drawn like any
   directed pair, its exocentricity is semantics, not structure). **Left-nested compounds are
   supported**: put the composite child on the left and the token on the right —
   `["B", "A", 2, 196]` — and the bracket draws MIRRORED (band running left, elbow
   left-then-up), as in the worked `(rāja-ṛṣi)-vaṃśaḥ` example. Sandhi note: print the
   **unsandhied segmented members** (plate convention) — `rāja-ṛṣi-vaṃśaḥ`, not the fused
   surface `rājarṣi-`.
3. **II часть start — ALWAYS ASK, never guess.** Ask which member *begins* II часть;
   everything from it rightward is II часть, everything left of it is automatically
   I часть. (The worked plate: II часть begins at `tamaḥ`.) "Omit the spans" is a valid
   answer — leave `II_START` out of the config and no I/II часть rules are drawn.
4. `label_x` per node are **plate parameters** — when reproducing a specific printed
   plate, measure them off the plate; when drawing fresh, place each label just right of
   its band end and nudge for even elbows.

## Config JSON

```json
{
  "tokens": ["uddāma", "ajñāna", "..."],
  "II_START": 4,
  "nodes": [["A", 6, 7, 1038], ["B", 5, 6, 873]],
  "title": "Klammerdiagramm",
  "style": {"FS": 39}
}
```

(`cx` may be added for the legacy spaced layout; omit it for the plate-true tight run.
`"ghosts": [{"after": 0, "text": "priya"}]` restores a madhyamapadalopin's elided
member in the word line in editorial brackets — `śāka-[priya]-pārthivaḥ` — no bar, no
node index (MG vote D4, 16-07-2026, H982 (Fable 5) — leitan-samasa-konspekt comparison; gallery §11).
`"trailing_hyphen": false` prints a complete word — e.g. a title like *abhijñāna-śākuntalam* —
instead of the plate's quoted-stem form with its trailing hyphen. Second committed worked
example, a two-member compound:
[klammerdiagramm-abhijnanasakuntalam-example.svg](https://github.com/gasyoun/SamasaChakram/blob/main/klammerdiagramm/klammerdiagramm-abhijnanasakuntalam-example.svg)
from [klammerdiagramm-abhijnanasakuntalam.json](https://github.com/gasyoun/SamasaChakram/blob/main/klammerdiagramm/klammerdiagramm-abhijnanasakuntalam.json).)

`style` overrides the plate constants (`FS`, `STEP`, `BW`, …) only when a target plate
demands it — the defaults ARE the traced plate.

## Verification

Headless-render the HTML and eyeball it (`msedge --headless=new --screenshot=... file:///...`);
when reproducing a printed plate, do a genuine side-by-side and list residual differences
honestly — visual 1:1 is judgment-gated, a human confirms. The reference plate is committed:
[klammerdiagramm-target-plate.png](https://github.com/gasyoun/SamasaChakram/blob/main/klammerdiagramm/klammerdiagramm-target-plate.png)
(MG's original upload, recovered from the H814 session transcript). Worked example (committed,
verified against MG's plate):
[klammerdiagramm-uddama-example.svg](https://github.com/gasyoun/SamasaChakram/blob/main/klammerdiagramm/klammerdiagramm-uddama-example.svg) ·
[klammerdiagramm-uddama-example.html](https://github.com/gasyoun/SamasaChakram/blob/main/klammerdiagramm/klammerdiagramm-uddama-example.html).

## Model tier — Sonnet 5 to draw, Opus 4.8 to decide contested nesting

The generator is deterministic: it owns all geometry, so plate fidelity never depends on the
model. The model's only job is the **analysis** → the `tokens` / `nodes` / `II_START` config,
plus asking the two mandatory questions (II часть start; ambiguous bracketing). That is a
bounded task, so **Sonnet 5 (claude-sonnet-5) is enough to DRAW** the common case — short-to-medium compounds with
obvious right-branching. Reach for **Opus 4.8 (claude-opus-4-8) or Fable 5 (claude-fable-5) to DECIDE contested constituency** —
left-nested sub-compounds, dvandva-of-dvandvas, long stanza ladders, or a scholar's disputed
reading (e.g. the uddāma D5 vote) — where a weaker model can pick a wrong split and the
generator will faithfully render the *wrong* analysis. The built-in guardrail (ALWAYS ask the
human to confirm ambiguous bracketing, never guess) is what makes Sonnet 5 safe to reuse; Opus 4.8
is the right default when reproducing a contested analysis. (Recorded per MG question, H997 (Opus 4.8) — leitan D1–D6 votes propagate.)

## Architecture call (recorded per H792 deliverable 5)

Standalone `/klammerdiagramm` — chosen over a mode of `/klammeruebersetzung` because the
deliverable differs (publication SVG artifact vs. woven translation prose); both consume
the same shared reference doc, so the analysis cannot drift. Census ruling U9
([FABLE_FOOTPRINT_SKILLS_CENSUS.md](https://github.com/gasyoun/Uprava/blob/main/archive/FABLE_FOOTPRINT_SKILLS_CENSUS.md) /
[H814 (Fable 5) — klammerdiagramm 1:1 plate fix](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H814-Fable_claude-config_klammerdiagramm-1to1-plate-fix_12.07.26.md))
had vetoed a standalone skill as a duplicate of the generator; MG overrode by ordering
[H792 (Fable 5) — klammerdiagramm skill 1:1](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H792-Fable_claude-config_klammerdiagramm-skill-1to1_12.07.26.md)
executed on 14-07-2026 (this file). U9's substance is honoured: the generator stayed
single-source in `references/assets/`, this command only drives it.

_Dr. Mārcis Gasūns_
