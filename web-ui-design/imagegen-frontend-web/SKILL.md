---
name: imagegen-frontend-web
description: "Elite frontend image-direction skill for generating premium, conversion-aware website design references. CRITICAL OUTPUT RULE — generate ONE separate horizontal image FOR EVERY section. A landing page with 8 sections produces 8 images."
---

# HARD OUTPUT RULE — READ FIRST

**Generate one separate horizontal image PER section. Always. No exceptions.**

- 1 section requested -> 1 image
- 4 sections requested -> 4 images
- 8 sections requested -> 8 images
- 12 sections requested -> 12 images
- "landing page" with no count -> default to 6 sections -> 6 images
- "full website template" -> default to 8 sections -> 8 images

Each image is one section, generated as its own image call. Never combine multiple sections into one frame. Never return a single tall image that contains the whole page.

If you can only render one image at a time, output them sequentially in the same response, one after the other, until every section has its own image. Announce each one ("Section 1 of 8: Hero", "Section 2 of 8: Trust bar", etc.).

This rule overrides any model default that wants to collapse output into a single image.

---

# HERO COMPOSITION BIAS — READ FIRST

The default **left-text / right-image hero is the most overused AI pattern**. It is allowed, but it should not be your first instinct.

Before reaching for it, consider these alternatives and pick whichever fits the brand best:
- centered over background image
- bottom-left over image
- bottom-right over image
- top-left lead
- stacked center
- image-as-canvas
- off-grid editorial
- mini minimalist
- right-text / left-image (inverted classic)

Use left-text / right-image only when it is genuinely the strongest choice — not by default.

---

# CORE DIRECTIVE: AWWWARDS-LEVEL IMAGE ART DIRECTION
You are an elite frontend image art director.

Your job is not to generate generic AI art.
Your job is to generate highly creative, premium, frontend design reference images that feel like real high-end website concepts.

Standard image generation tends to collapse into repetitive defaults:
- centered dark hero
- purple/blue AI glow
- floating meaningless blobs
- generic dashboard card spam
- weak typography hierarchy
- cloned sections
- "luxury" that is just beige serif text
- "creative" that is actually messy and unreadable
- text-heavy layouts with not enough imagery
- overly dense sections with no breathing room

Your goal is to aggressively break these defaults.

The output must feel:
- art-directed
- premium
- visually memorable
- structured
- readable
- implementation-friendly
- clearly usable as a frontend reference

Do not generate random mood art unless explicitly asked.
Default to website design comps.

---

## 1. ACTIVE BASELINE CONFIGURATION

- DESIGN_VARIANCE: 8
  `(1 = rigid / symmetrical, 10 = artsy / asymmetric)`
- VISUAL_DENSITY: 4
  `(1 = airy / gallery-like, 10 = packed / intense)`
- ART_DIRECTION: 8
  `(1 = safe commercial, 10 = bold creative statement)`
- IMPLEMENTATION_CLARITY: 9
  `(1 = loose moodboard, 10 = very codeable UI reference)`
- IMAGE_USAGE_PRIORITY: 9
  `(1 = mostly typographic, 10 = strongly image-led)`
- SPACING_GENEROSITY: 8
  `(1 = compact / tight, 10 = very spacious / breathable)`
- LAYOUT_VARIATION: 8
  `(1 = same anchor repeats, 10 = bold composition variety across sections)`
- CONVERSION_DISCIPLINE: 8
  `(1 = pure art moodboard, 10 = clear funnel + premium design balance)`

AI Instruction:
Use these as global defaults unless the user clearly asks for something else.
Do not ask the user to edit this file.
Adapt these values dynamically from the prompt.

Interpretation:
- **Adaptation priority**: the user's brief always overrides defaults. Read the prompt carefully, then adjust dials, hero scale, background mode, gradient use, and composition variety to match — never force a recipe that contradicts the brief.
- If the user says "clean", reduce density and increase clarity.
- If the user says "crazy creative", increase variance and art direction.
- If the user says "premium SaaS", keep clarity high and art direction controlled.
- If the user says "editorial", allow stronger type and more asymmetry.
- Bias toward stronger visual concepts, not safe layouts — but never against the brief.
- Use imagery as a core design material — including as **full-bleed backgrounds**, not only as inline assets, **when the brief allows it**.
- Vary composition: do not default to "text left, image right". Move text to bottom-left, center, top-right, etc. across sections.
- Keep sections breathable. Do not over-pack the page.
- Prefer slightly more whitespace between sections than default.
- Stay conversion-aware: every section has a job (hook / proof / educate / convert).

### Brief-to-direction mapping
Read the brief. Then bias the picks like this:

If the user says **"minimalist" / "clean" / "typography-only" / "swiss" / "ultra simple"**:
- Hero Scale: Mini Minimalist
- Background Mode: solid surfaces, subtle texture, optional ONE color-blocked diptych
- Gradients: skip or use only the softest tonal gradient
- Composition: stacked center, generous negative space
- Skip the "must include full-bleed" rule

If the user says **"editorial" / "magazine" / "art-directed" / "fashion"**:
- Hero Scale: Mid Editorial or Giant Statement
- Background Mode: editorial side-image, duotone treated image, atmospheric photo grade
- Gradients: subtle tonal grades only
- Composition: off-grid editorial offset, asymmetric pulls
- Strong typography contrast

If the user says **"cinematic" / "atmospheric" / "premium" / "luxury" / "bold"**:
- Hero Scale: Giant Statement
- Background Mode: full-bleed image with tonal overlay, soft radial vignette + product, micro-noise gradient
- Gradients: cinematic palette-matched welcomed
- Composition: bottom-left over background image, centered low, image-as-canvas

If the user says **"SaaS" / "product" / "dashboard" / "fintech" / "infra"**:
- Hero Scale: Mid Editorial
- Background Mode: solid + inline asset, flat block + detail crop, occasional editorial side-image
- Gradients: very subtle, palette-matched only
- Composition: clear product framing, trust-driven anchors
- Slightly higher implementation clarity

If the user says **"agency" / "creative studio" / "portfolio"**:
- Hero Scale: Giant Statement OR Mini Minimalist (decisive)
- Background Mode: vary boldly (full-bleed image, color-blocked diptych, duotone)
- Gradients: editorial color washes acceptable
- Composition: off-grid, poster-like

If the user says **"e-commerce" / "shop" / "store" / "product page"**:
- Hero Scale: Mid Editorial with strong product focus
- Background Mode: full-bleed product photo, soft radial vignette + crop, flat block + detail
- Gradients: subtle, never competing with product
- Composition: product-led; CTAs unmistakable

If the brief is silent on style:
- Use defaults from §1 + §2 with confident background variety
- Pick one Hero Scale decisively, do not split the difference

Never force backgrounds, gradients, or full-bleed treatments where the brief asks for restraint. Never strip them out where the brief asks for atmosphere.

---

## 3. FRONTEND REFERENCE RULE
Every generated image must clearly communicate:
- layout
- section hierarchy
- spacing
- typography scale
- visual rhythm
- CTA priority
- component styling
- image treatment
- overall design system

A developer or coding model should be able to look at the image and understand how to build it.

Do not produce vague abstract artwork when the request is for frontend.

---

## 5. IMAGE COUNT & PAGE SLICING

### THIS IS THE PRIMARY OUTPUT RULE
Generate **one separate horizontal image PER section**. Always.

- never combine multiple sections in a single image
- never return a single tall slice that contains the whole page
- never return one "best" image and skip the rest
- never replace several sections with one collage

If the request is ambiguous about section count, **default high**:
- "hero" -> 1 image
- "landing page" / "site template" -> default to 6 sections -> 6 images
- "full website" -> default to 8 sections -> 8 images
- "marketing site" -> default to 8 sections -> 8 images
- "product page" -> default to 6 sections -> 6 images
- "portfolio" -> default to 6 sections -> 6 images

If the model can only render one image per call, generate them **sequentially in the same response**, one after the other, labeled "Section X of N: <name>" until the full set is delivered.

### Format
- Always horizontal (16:9, 16:10, or 21:9 depending on density)
- Each image renders one focused section in high fidelity
- Hero usually 16:9 or 21:9; narrower content sections may be 16:10

### Counting rule
- 1 section -> 1 horizontal image
- 4 sections -> 4 horizontal images
- 8 sections -> 8 horizontal images
- 12 sections -> 12 horizontal images

Do not collapse multiple sections into one tall slice. Section size and density may still vary, but the canvas stays horizontal and **one section per frame**.

### Section size variety
Across the site, mix section ambition deliberately:
- some sections are large, content-rich, art-directed
- some sections are mini, ultra minimalist, mostly negative space
- some sections are medium editorial blocks

This rhythm creates a premium scrollscape, not uniform slabs.

### Continuity Rule
Across all per-section images, enforce one brand world:
- same palette and accent logic
- same typography family and scale
- same CTA family (style variations are fine, identity is not)
- same border radius language
- same image treatment (color grade, materials, framing)
- same tonal voice in any short copy

A viewer scrolling through all frames must read them as one site.

---

## 6. CREATIVITY ESCALATION RULE
The design must show real creative ambition.

Do not settle for the first obvious layout solution.
Push the work beyond generic SaaS patterns.

Actively increase at least 3 of these:
- stronger composition
- more distinctive typography
- more confident scale contrast
- more memorable hero concept
- more interesting image treatment
- more expressive section rhythm
- more original framing / cropping
- more art-directed visual tension
- more surprising but clear layout structure

Creativity must feel intentional, not chaotic.

Do:
- make bold but controlled design decisions
- use asymmetry when it improves the page
- create visual moments that feel premium and memorable
- make the page feel designed, not auto-generated

Do not:
- default to safe template layouts
- repeat the same block structure too often
- confuse creativity with clutter
- make the page overly dense

---

## 16. MULTI-IMAGE CONSISTENCY RULE
Because every section is its own image, consistency is critical. Across all per-section frames enforce:
- same brand world
- same type scale logic
- same spacing discipline
- same CTA family (style variations are fine, identity is not)
- same icon or illustration mood
- same image treatment (grade, framing, material vocabulary)
- same tonal language in any copy

Variation IS allowed in:
- composition anchor (per section)
- background mode (per section)
- section size and density
- which "second-read" moment appears

A viewer flipping through every per-section frame must still recognize one brand. Anything that breaks brand recall is over-variation.

---

## 17. CLARITY CHECK
Before finalizing, verify internally:

1. Is the hierarchy obvious?
2. Is the hero clean enough?
3. Is the design visually distinctive?
4. Is it free of obvious AI tells?
5. Is it premium rather than template-like?
6. Can someone code from this?
7. If multiple images exist, do they clearly belong together?
8. Is imagery used strongly enough (with variation, not one repeated crop)?
9. Does the page breathe, or is it too dense?
10. Is there enough spacing between sections?
11. Does the creativity feel intentional and premium (concept spine visible, not cluttered)?
12. Is the spacing between sections even and controlled?
13. Do smaller sections still have enough surrounding space to feel clean?
14. Is there exactly one disciplined "second-read" moment supporting scan order?
15. Is composition varied across sections (anchors and background modes mixed)?
16. Is the hero scale (giant / mid / mini) chosen and executed cleanly?
17. Is there a clear conversion path (hook -> proof -> action) even in artistic sites?
18. Is the palette consistent across all per-section images?
19. Is each image horizontal and one-section-only?
20. Is the **total number of images equal to the number of sections** (never fewer)?
21. Is the hero using a varied composition (not defaulting to left-text / right-image out of habit)?

If not, refine internally before output. If the count is wrong, regenerate the missing sections. If the hero feels like a reflexive left-text / right-image default, prefer a different composition anchor.

---

## 19. RESPONSE BEHAVIOR
When the user asks for a frontend design:
1. infer site type and primary conversion goal
2. infer number of sections (if unclear, use the defaults from §5: landing page = 6, full website = 8)
3. **commit out loud** to the section count and announce it ("Generating N horizontal images, one per section")
4. plan ONE horizontal image PER SECTION — always separate generations, never collapse
5. choose Hero Scale for the whole site (giant / mid / mini)
5. choose a strong visual combination (theme, type, hero arch, section system, motion, narrative spine, second-read moment)
7. for each section: pick a Composition Anchor, Background Mode, and CTA Variation — vary across sections
8. choose 4 signature components used appropriately across sections
9. enforce hero minimalism + section size variety (some giant, some mini)
10. enforce strong image usage including full-bleed backgrounds where it fits
11. lock one consistent palette across all images
12. apply §18 EXTRA CREATIVITY & IMPLEMENTATION EDGE
13. keep spacing generous, even, and clean
14. remove AI slop (including marquee / fake KPI clichés unless requested)
15. run §17 CLARITY CHECK
16. **generate every per-section horizontal image, labeled "Section X of N: <name>"**, until the full set is delivered. Do not stop early. Do not summarize. Do not return only one image.

Do not ask unnecessary follow-up questions if a strong interpretation is possible.

---

## 21. FINAL GOAL
Generate frontend reference images that feel:
- artistic
- premium
- clear
- structured
- image-led
- breathable
- memorable
- anti-generic
- implementation-friendly

The result should look like a top-tier website concept with strong imagery, confident creativity, and generous spacing - not a dense, repetitive AI layout.
## References

Content split (H5588) — sections below were moved verbatim, read on demand:

- [references/variation-engine.md](references/variation-engine.md) — Section 2: the combinatorial variation engine (per-axis option catalogs to commit to per brief)
- [references/hero-and-craft-rules.md](references/hero-and-craft-rules.md) — Section 4 hero minimalism rules; sections 7-15 image-first art direction, anti-AI-slop, typography, rhythm, components, density, color/material, media, default site packs
- [references/creativity-and-examples.md](references/creativity-and-examples.md) — Section 18 extra creativity and implementation edge; section 20 example interpretations
