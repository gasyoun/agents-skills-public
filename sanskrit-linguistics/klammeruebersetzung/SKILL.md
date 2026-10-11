---
name: "klammeruebersetzung"
description: 'Weave a Sanskrit source and a separate Russian/English (+German) translation into a Klammerübersetzung - each target word carries its source word in parentheses right after it. Triggers: "weave", "bracket translation"; tatpuruṣa/dvandva unpacking.'

---

# klammeruebersetzung

Use this skill when the user asks to run the migrated source command `klammeruebersetzung`.

## Command Template


# /klammeruebersetzung

_Created: 12-07-2026 · Last updated: 26-08-2026_

**Klammerübersetzung** ("bracket / interlocking translation") is the standard device of the
German Indological school — the Petersburg dictionary tradition this project descends from
(Böhtlingk-Roth, PWG/PW) — for making a dense Sanskrit passage transparent without giving up
natural target-language word order. The translation reads normally in Russian / German /
English; **after each translated element its Sanskrit source word stands in parentheses,
italicised**. The reader sees the correspondence word-for-word and can check every choice.

For a **compound or a vigraha (analytic paraphrase)** the skill also does the thing a naive
left-to-right gloss cannot: it resolves the structure **head-first**. In a Sanskrit
determinative compound (and in a well-formed vigraha sentence) the syntactic head is the
**last** word, so the analysis runs **right → left**. You name the head first, then ask "which
one? / made how? / referring to what?" and peel one modifier at a time leftwards, and only
then read the accumulated brackets back left-to-right as the smooth translation.

## Input contract

The user supplies the two sides **separately** — do not expect them pre-aligned:

- **Sanskrit** — a compound (`abhijñānaśākuntalam`), a vigraha sentence
  (`abhijñānaṃ ca śākuntalā cety … kṛtaṃ nāṭakam`), or a short passage. IAST in, IAST out;
  never re-transcode silently (if a transcode is needed, say so — ITRANS→SLP1 via
  `indic_transliteration.sanscript`, per memory).
- **Translation** — Russian **or** English (the primary target), optionally **plus German**
  as a second bracketed layer.

If only the Sanskrit is given, produce the analysis and *propose* the translation, flagging it
as yours to be checked — do not present a generated rendering as the user's.

## Output — two blocks, in this order

### 1. The right-to-left resolution ladder (only for compounds / vigraha)

The right→left case-question mechanism is documented canonically in
[`sanskrit-right-to-left-reading.md` (SamasaChakram)](https://github.com/gasyoun/SamasaChakram/blob/main/sanskrit-right-to-left-reading.md)
(shared with [`/sanskrit-parse`](https://github.com/gasyoun/claude-config/blob/main/commands/sanskrit-parse.md), the parse-only entry point). This skill
runs that same ladder and then weaves the bracket translation on top of it.

**The question IS the mechanism.** Each leftward step is not "add the next word" but "ask the
question the current word leaves open, and let the next word answer it." Name the head, then:
*какая? → как? → кого/чего? → каких именно? → а что это значит?* Every rung is
question-driven; the accumulating brackets are just the by-product.

Number the steps from the **head** (rightmost word) leftwards. Each rung shows the accumulated
Sanskrit tail, the word being added + gloss, and — first — the **driving question** it answers:

```
① nāṭakam                     — что?              → драма (nāṭakam)
② kṛtaṃ nāṭakam               — какая?            → созданная (kṛtam)
③ sadhikṛtya kṛtaṃ …          — как? каким образом? → сделав предметом (sa+adhikṛtya)
④ te sadhikṛtya …             — сделав предметом кого? → этих двух (te, вин. дв. ч.)
⑤ abhijñānaśākuntale te       — этих двух — каких? → «Абхиджняна-Шакунтала» (abhijñānaśākuntale)
⑥ …ca …ca iti (cety)          — а что это значит? → маркер называния (iti)
⑦ abhijñānaṃ ca · śākuntalā ca — какие два?        → узнавание (abhijñānam) и Шакунтала (śākuntalā)
```

Then emit two condensed by-products of the ladder:

- **The bare question chain** — the driving questions alone, so the reading logic is visible in
  one line: `Что? → Какая? → Как? → Кого? → Каких? → А что это значит?`
- **The cascading unfolding** — each answer indented under the question it settles, ending in the
  dvandva members. This is the ladder read as a single descent.

State once, at the top, **why** the direction is right-to-left ("head = last word, so we peel
modifiers leftward"). Mark dvandva members, the naming particle `iti`, sandhi splits
(`sadhikṛtya = sa + adhikṛtya`, `cety = ca + iti`) explicitly — a swallowed sandhi hides a step.
Where morphology *disambiguates a case*, say so and use it (see the `te`/`-e` note in the worked
reference: a gerund like `adhikṛtya` governs the **accusative**, and `te` / `…e` are nom/acc dual,
not locative — `tayoḥ` / `…layoḥ` would be locative). Only park a reading as ambiguous when the
forms genuinely allow both.

### 2. The woven Klammerübersetzung

Read the ladder back **left-to-right** as one natural target-language sentence, each chunk
carrying its source word in italic parentheses. Two-target form stacks German after Russian:

> драма (*nāṭakam*, dt. *Drama*), созданная (*kṛtam*) …

Keep the target sentence grammatical in the target language; the brackets annotate, they do not
dictate word order. Reserve `[…]` square brackets for words you supply that have no source word
(implied "[предмета]", "[пару]"); parentheses are strictly for the source word.

## Worked reference — `abhijñānaśākuntalam`

Vigraha: `abhijñānaṃ ca śākuntalā cety abhijñānaśākuntale te sadhikṛtya kṛtaṃ nāṭakam`
— "the drama composed with reference to these two, recognition and Śakuntalā".

Ladder (head-first, right→left — question first, answer second):

```
① nāṭakam                                   — что?               → драма (nāṭakam)
② kṛtaṃ nāṭakam                             — какая?             → созданная/сочинённая (kṛtam)
③ sa+adhikṛtya (sadhikṛtya) kṛtaṃ nāṭakam   — как? каким образом? → сделав предметом (adhikṛtya)
④ te sadhikṛtya …                           — сделав предметом кого? → этих двух (te, вин. дв.ч.)
⑤ abhijñānaśākuntale te …                   — этих двух — каких?  → «Абхиджняна-Шакунтала» (abhijñānaśākuntale)
⑥ …ca …ca iti (cety)                        — а что это значит?   → маркер называния (iti)
⑦ abhijñānaṃ ca · śākuntalā ca              — какие два?          → узнавание (abhijñānam) и Шакунтала (śākuntalā)
```

Bare question chain: **Что? → Какая? → Как? → Кого? → Каких? → А что это значит?**

Cascading unfolding:

```
драма (nāṭakam)
└ какая? созданная (kṛtam)
  └ как? сделав предметом (adhikṛtya)
    └ кого? этих двух (te)
      └ каких двух? «Абхиджняна-Шакунтала» (abhijñānaśākuntale)
        └ то есть: узнавание (abhijñānam) и Шакунтала (śākuntalā)
```

Case note (morphology disambiguates): `adhikṛtya` is a gerund governing the **accusative** —
"X-am adhikṛtya" = "having made X the topic". `te` and `abhijñānaśākuntale` end in `-e`, the
nom/acc dual; the locative dual would be `tayoḥ` / `abhijñānaśākuntalayoḥ`. So these are
**accusative dual** (the direct object of the gerund), *not* locative — even though the sense the
gerund delivers is "concerning / об этих двух". Read the *meaning* as "concerning", but label the
*case* accusative.

Woven: **драма (*nāṭakam*), созданная / сочинённая (*kṛtam*), сделав предметом (*adhikṛtya*)
этих двух (*te*, вин. дв. ч.) — то есть (*iti*) [пару] «узнавание-и-Шакунтала»
(*abhijñānaśākuntale*): узнавание (*abhijñānam*) и Шакунталу (*śākuntalā*).**

## Scope / caveats

- The ladder is an *analytic aid*, not a claim about compound type — say whether the resolution
  reads it as tatpuruṣa / dvandva / bahuvrīhi, and where the vigraha is ambiguous, give both
  readings rather than silently picking one.
- Never invent the source-side word for a bracket: if a Russian word answers to no single
  Sanskrit word (a case-ending nuance, an implied copula), mark it `[…]`, don't parenthesise it.
- Devanagari/IAST I/O needs a UTF-8 console.

_Dr. Mārcis Gasūns_
