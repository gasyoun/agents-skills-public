## Gotchas (learned on PWG + later runs)

- Reading a downsampled full page → fluent but fabricated text. Always crop to native resolution. This is the #1 rule.
- Crop ≤1900 px/side; never upscale past 2000 px (hard API limit; killed several subagents).
- Trust the toctree order, not the image-filename sort.
- A foreword's place/date/signature may be split across the column foot (PWG vol.5 had **no** date; vol.7's date sat under the left column, names under the right). Don't invent a date.
- Don't commit temp crops (the first PWG commit accidentally swept in ~180 of them; had to be stripped before push).
- RU: Cyrillic names, no redundant Latin parens; keep work-titles in original script.
- Source ≠ German for most dicts — detect language; produce `.ru` always, `.en` only when source isn't English.
- **MWE lives in `MWS`**, not a separate repo; **WIL + YAT** share `Wil-YAT`. **PW** → repo `PWK`. **IEG** may land in staging `prefaces_ieg` until a home repo exists.
- **Do not re-OCR Done rows** — check `prefaces/README.md` + `*pref_all.*` first.
- **PD / `feat/ocr-v2-pipeline` is a different corpus** (Deccan College encyclopaedic dictionary PDF front matter). Reuse its crop-then-OCR ideas for Phase 3.5 audit and for PDF tables; do not replace vision OCR of csldoc PNGs with Tesseract as author. Residual Tesseract noise still needs vision repair for scholarly editions.
- **Phase 3.5 false positives are normal** on abbrev keys and title-page names — flag only coverage/layout/number anomalies, not every diacritic mismatch.
- Empty local `prefaces/` dirs (`BUR`, `SKD`, `PE`, `PUI`) mean "started or stubbed", not done — treat as queue.
- **Phase 8 is not re-OCR** — if you re-transcribe Done pages instead of running 8a–8i, you are on the wrong skill path.

