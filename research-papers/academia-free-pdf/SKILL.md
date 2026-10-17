---
name: "academia-free-pdf"
description: Download a free PDF from academia.edu through MG's logged-in Chrome via ZCode computer-use — right tab "PDF only" → left "stay on the free tier"; the anonymous in-app browser only reaches the signup wall.
---

# academia-free-pdf

_Created: 05-10-2026 · Last updated: 05-10-2026_

Use when a paper's PDF lives on academia.edu and the file must land in `~/Downloads`.
Proven 05-10-2026 on three papers in one session (Shakya 2019, Majcher 2023 ACSA,
Leonchenko lexical cores) — [H6122](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H6122-GLM_Uprava_academia-free-pdf-batch_05.10.26.md),
files landed in [Uprava/papers/refs](https://github.com/gasyoun/Uprava/blob/main/papers/refs/README.md).

## Preconditions

- **Backend: computer-use on MG's Mac.** The browser-use in-app browser fails: its session
  is anonymous, «Download Free PDF» only opens the «Continue with Google» wall, and direct
  HTTP without a fresh session token gets 403 (anonymous) / 404 (stale token) — the token
  is minted with the page, it cannot be constructed outside a real browser session.
- Chrome running, logged into academia.edu as Mārcis. Free tier is enough — never accept
  the €1 pack.

## Steps

1. **Find the work URL:** one `spider_search` / `WebSearch` with
   `site:academia.edu <author> <title fragment>` — take the single hit, never guess URL
   variants or numeric IDs.
2. **Open it in Chrome:** ⌘L → paste URL → Return. First load may park on a Cloudflare
   «Just a moment...» for ~10 s — re-observe before acting, do not reload into it.
3. **The download dialog auto-opens** on work pages. THE DEFAULT TAB IS THE TRAP: left tab
   «Full PDF Pack for €1» = premium bundle of 37+ papers. Click the RIGHT tab
   **«PDF only»** (AX radio whose name contains `PDF only`).
4. Click **«Download PDF →»** under «Get this PDF».
5. Modal «Are you sure you don't want the PDF Pack?» → click the LEFT button
   **«No thanks, I'll stay on the free tier»** — the download starts
   («YOUR DOWNLOAD HAS STARTED»).
6. Second modal «Get 20 more free papers... Download the free Library app» → **«No thanks»**.
   Never touch «Download PDF Pack for €1» or «Download free Library app».

## Gotchas

- Playwright locator clicks time out on academia.edu buttons (actionability never settles).
  Drive it with computer-use element indices from a fresh AX tree, or CUA coordinates off
  the current screenshot — one state-changing action per observation cycle.
- The Chrome toolbar download arrow is the cheapest completion check
  («&lt;name&gt;.pdf • Done»); confirm the artifact with
  `file ~/Downloads/<name>.pdf` (page count).
- Disk names are title-truncated, not author-based:
  `The_Digital_Sanskrit_Buddhist_Canon_Proj.pdf` = Shakya 2019,
  `Whose_Veda_Whose_Story_Reflections_on_di.pdf` = Majcher 2023.
- Several papers per session: reuse the same tab (⌘L → new URL); each work page re-opens
  its own dialog.
- Filing: reference batches without an A-ID go to
  [Uprava/papers/refs](https://github.com/gasyoun/Uprava/blob/main/papers/refs/README.md)
  with a provenance manifest (source URL, attachment id, page count, executor); move the
  file into `papers/<A-ID>_<venue>/` when the owning paper activates.

_Dr. Mārcis Gasūns_
