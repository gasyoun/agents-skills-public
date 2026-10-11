## 8. Summary Card Pattern

Don't use 3 identical generic cards. Vary the treatment:

```html
<div class="card">
  <p class="eyebrow">SECTION LABEL</p>
  <div class="card-header">
    <span class="card-dot coral"></span>
    <h3>Card Title</h3>
  </div>
  <ul><li>Item</li></ul>
</div>
```

Rules:

- `background: #ffffff` (not paper — slight lift without shadow)
- `border: 1px solid rgba(45,49,66,0.12)`
- `border-radius: 6px`, `padding: 1.25rem`
- **No `box-shadow`**
- Card dots: 7px, `border-radius: 50%` — ink / muted / coral / link / soft variants

---

## 10. Templates & Variants

Every diagram ships in three variants (see `assets/`):

| Variant | File pattern | When to use |
|---|---|---|
| **Minimal light** (default) | `assets/template.html`, `example-<type>.html` | Screenshot-ready. Diagram + title. Warm paper. |
| **Minimal dark** | `assets/template-dark.html`, `example-<type>-dark.html` | Dark mode sites, slides, high-contrast posts. |
| **Full editorial** | `assets/template-full.html`, `example-<type>-full.html` | Long-form posts where the diagram is the hero. |
| **Consultant special** (quadrant only) | `example-quadrant-consultant.html` | BCG/McKinsey-style 2×2 scenario matrix. Clinical sans-serif, white bg, bold blue double-ended axes, named scenario cells. See [type-quadrant.md](references/type-quadrant.md#consultant-special-2x2-scenario-matrix). |

**Sketchy variant** (optional, applied to any of the above) — see [primitive-sketchy.md](references/primitive-sketchy.md). SVG turbulence filter wobbles strokes for a hand-drawn feel. Good for essays, not for technical docs.

**Terminal variant** (optional, replaces any of the above) — see [primitive-terminal.md](references/primitive-terminal.md). Start from `assets/template-terminal.html`; terminal examples use the `example-<type>-terminal.html` naming pattern. Charcoal CLI-window chrome, monospace, one red-orange accent. Good for dev-tool posts; not brand-tokenized, so skip it for onboarded output.

**Animation** (optional presentation layer) — see [animation.md](references/animation.md). Modes are `none` (default), `reveal`, `step`, and `loop`; motion never changes the static meaning or raises the complexity budget.

### To create a new diagram

1. Copy the variant closest to what you want (`assets/template.html` for minimal, `assets/template-full.html` for cards, `assets/template-motion.html` only when motion is requested).
2. If behavior is load-bearing, choose a semantic pattern; then load the matching type reference linked in the visual-type guide.
3. Replace the eyebrow, h1, and SVG body. Replace `[diagram-slug]` with the file slug and fill `<title>` / `<desc>`.
4. If motion is requested, load `animation.md`; otherwise keep mode `none` and no script.
5. Run the §9 taste gate.

---

### Accessible SVG contract

Every diagram is an accessible figure by default:

1. Its `<svg>` carries `role="img"` and `aria-labelledby` naming the diagram's `<title>` and `<desc>`.
2. `<title>` is the first child of `<svg>`, before `<defs>`. Assistive technology may ignore a title placed later.
3. The IDs are prefixed per diagram and variant: `<slug>-title` / `<slug>-desc`, where the slug matches the file (`loop`, `loop-dark`, `loop-full`). Bare `title` / `desc` IDs are banned because two inline diagrams would create duplicate IDs and the second could be announced with the first diagram's name.
4. `<title>` is the short name of the subject — roughly the page `<h1>`, and about 60 characters or fewer.
5. `<desc>` is one sentence stating what the diagram shows in terms a reader needs without the image. Describe the content, not the geometry: “Org chart showing a command center routing work to specialist agents and escalation owners,” not “A box at the top with five boxes below it.” A shape-by-shape narration is worse than no useful description.
6. Decorative-only SVG, such as the specimen glyphs in `assets/icons.html`, carries `aria-hidden="true"` instead. Giving decorative marks accessible names adds noise.

### Exporting to PNG / SVG

When the user asks to export, save, rasterize, or convert a generated diagram to `.png` or `.svg`, load [`references/export.md`](references/export.md) and follow the procedure there. Both formats deliver the diagram only (the `<svg>` node) — editorial wrappers like cards and headers are dropped by design. Export is **manual** — never produce export files unprompted.

For an imported diagram, pixel dimensions come from the `viewBox` × scale factor, so its size decision belongs to §11, not to export. For any diagram that needs an exact frame (an OG card or a 1920×1080 slide image), see [`export.md` § Sizing the export](references/export.md).
