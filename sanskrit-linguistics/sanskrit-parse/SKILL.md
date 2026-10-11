---
name: "sanskrit-parse"
description: 'Parse a Sanskrit sentence/compound right-to-left by case-questions - split sandhi, morphology settles each case; comprehension parse, not a bracket translation. Triggers: "разбери это предложение". Gloss-weave route: /klammeruebersetzung.'

---

# sanskrit-parse

Use this skill when the user asks to run the migrated source command `sanskrit-parse`.

## Command Template


# /sanskrit-parse

_Created: 12-07-2026 · Last updated: 26-08-2026_

Applies the canonical **case-question, right-to-left reading method** —
[`sanskrit-right-to-left-reading.md` (SamasaChakram)](https://github.com/gasyoun/SamasaChakram/blob/main/sanskrit-right-to-left-reading.md)
— to a Sanskrit sentence or compound the user supplies, and returns a **step-by-step parse for
comprehension**. This is the *teaching / understanding* entry point; the mechanism is shared
with `/klammeruebersetzung`, which additionally produces the
bracket translation. This skill is deliberately **not** an automated morphological parser (vidyut
/ DCS etc.) — it is the human reading discipline, applied and explained.

## Input

- **Sanskrit only** (IAST): a sentence, a vigraha paraphrase, or a compound. No translation is
  required — that is the point; you parse to *understand*, then optionally translate.

## Method — do not restate the reference, execute it

Follow the reference's Step 0→5 exactly. In brief, right→left:

0. **Anchor** — the last word: a Nom. noun or a finite/absolute verb. → *что?*
1. **Agreeing modifier** — participle / adjective / numeral. → *какая?*
2. **Gerund / adverbial** (`-tvā` / `-ya`). → *как? каким образом?*
3. **Object** of that action. → *кого? что?* (watch number)
4. **Unpack the object**. → *каких именно?*
5. **Resolve `iti`-marked names** (mind `cety = ca + iti`). → *что это значит?*

## Output — three parts (+ an optional rendering)

1. **Sandhi split** of anything fused (`sadhikṛtya = sa + adhikṛtya`, `cety = ca + iti`) — before
   counting steps, so no rung is hidden.
2. **The numbered ladder**, head-first, each rung: *question → answer (word, gloss, case/number)*.
   State the anchor and why the read runs right-to-left once, at the top.
3. **The bare question chain** on one line — e.g. `Что? → Какая? → Как? → Кого? → Каких? → Что значит?`
   — the reading logic made visible.
4. **On request — the staircase rendering.** The same parse drawn as ASCII, three forms (all in the
   reference's *Optional rendering* section): **(1)** descending staircase **with questions**
   (teaching), **(2)** compact staircase **without questions, with glosses** (readable structural
   view), **(3)** ultra-compact **bare** (words + connectors only, tersest). It is a *view*, not a
   new analysis. **The head is always rightmost — never render it on the left** (it is the last
   word; a head-left layout inverts the structure). Fixed-width only; the fork sits at the dvandva,
   `[iti]` marks the naming node. For a long multi-member compound, the publication-grade form is
   the **Klammerdiagramm (SVG)** — hatched arrow-topped brackets under each member + a descending
   staircase of binary nodes; see the reference's *Publication-grade form* section and its committed
   worked example. Produce it as SVG (`currentColor`, hatch `<pattern>`, arrow `<marker>`), not ASCII.

## Hard rules (from the reference)

- **No question, not done.** A word that answers no question yet is unattached — keep going; never
  translate it in isolation.
- **Morphology settles the case.** A gerund (`adhikṛtya`) governs the **accusative**; a dual in
  `-e` is Nom./Acc., not locative (`tayoḥ`/`…layoḥ` would be locative). Read the sense, label the
  case correctly; park a word as ambiguous only when its form truly allows two parses.
- IAST in, IAST out — never silently re-transcode.

## Handoff

If the user then wants the woven, source-in-parentheses rendering (for a dictionary entry or a
paper), pass the same sentence to `/klammeruebersetzung` — it
reuses this exact ladder and adds the bracket translation.

_Dr. Mārcis Gasūns_
