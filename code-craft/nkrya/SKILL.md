---
name: "nkrya"
description: 'NKRYa (Russian National Corpus) evidence for a Russian word or collocation choice — word sketch (top-10 collocates, dice), ipm frequency, pair concordance with a 19th-century slice, via the official ruscorpora.ru API client. Advisory evidence card; a human votes, nothing is applied. Sonnet 5 (claude-sonnet-5). Triggers: "/nkrya", "check in НКРЯ", "what does the corpus say about тучи сгущаются". Do not use to scrape ruscorpora.ru or to auto-apply a rendering.'
---

# nkrya

Use this skill when the user asks to run the migrated source command `nkrya`.

## Command Template


# /nkrya <lemma[:POS]> [head[:POS]] | --card <id>

_Created: 23-09-2026 · Last updated: 23-09-2026_

Thin wrapper over the PWG-RU client [nkrya_client.py](https://github.com/gasyoun/SanskritLexicography/blob/master/RussianTranslation/src/nkrya_client.py) and card builder [nkrya_evidence_card.py](https://github.com/gasyoun/SanskritLexicography/blob/master/RussianTranslation/src/nkrya_evidence_card.py) (H5261). Rulings: [GRILL_NKRYA_SKILL_DECISIONS_22-09-2026.md](https://github.com/gasyoun/SanskritLexicography/blob/master/RussianTranslation/docs/GRILL_NKRYA_SKILL_DECISIONS_22-09-2026.md).

## Procedure

Run from `SanskritLexicography/RussianTranslation`.

1. **Token first.** `python src/nkrya_client.py probe` — it checks the token and whether the 19th-century subcorpus filter is accepted. No token → the client prints the storing command; hand it to MG and stop. Never paste a key into chat, git or a repo `.env`.
2. **One word:**

   ```bash
   python src/nkrya_client.py freq сплочённый A
   ```

   ```bash
   python src/nkrya_client.py sketch туча S
   ```

3. **A pair** (candidate within 3 words of the head, either order; add `--19c` for the archaism check):

   ```bash
   python src/nkrya_client.py pair сгущаться туча -n 3
   ```

4. **An evidence card** for a correction row — candidates as `surface=lemma:POS` (NKRYa lemmatizes participles to the verb):

   ```bash
   python src/nkrya_evidence_card.py --id C07 --heads туча:S,облако:S --candidates связный:A,сплочённый:A,сгущающийся=сгущаться:V --out pwg_ru/h5069/C07_NKRYA_EVIDENCE_CARD.md --json-out pwg_ru/h5069/C07_NKRYA_EVIDENCE_CARD.json
   ```

5. **Commit the evidence used**: the card plus exactly the cache entries it lists under `pwg_ru/nkrya_cache/`.
6. **Put it to MG as a vote.** The card is advisory; it never changes a TSV `vote_*` value or the store.

## Reading the numbers

1. A sketch lists only the **top 10** collocates per relation. "Not in top 10" is not "unattested"; the pair count is the attestation.
2. ipm category 1–6 is NKRYa's band (1 = under 1 ipm … 6 = over 10 000).
3. Offline (`--offline`) reads only the cache — use it in CI and for re-rendering.

_Гасунс_
