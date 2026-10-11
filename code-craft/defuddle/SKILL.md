---
name: defuddle
description: 'Extract clean markdown from web pages via Defuddle CLI, removing clutter/navigation to save tokens. Use instead of WebFetch when given a URL to read/analyze. NOT for .md URLs — already markdown, WebFetch directly.'

---

_Created: 03-07-2026 · Last updated: 29-08-2026_


# Defuddle

Use Defuddle CLI to extract clean readable content from web pages. Prefer over WebFetch for standard web pages — it removes navigation, ads, and clutter, reducing token usage.

If not installed: `npm install -g defuddle`

## Usage

Always use `--md` for markdown output:

```bash
defuddle parse <url> --md
```

Save to file:

```bash
defuddle parse <url> --md -o content.md
```

Extract specific metadata:

```bash
defuddle parse <url> -p title
defuddle parse <url> -p description
defuddle parse <url> -p domain
```

## Output formats

| Flag | Format |
|------|--------|
| `--md` | Markdown (default choice) |
| `--json` | JSON with both HTML and markdown |
| (none) | HTML |
| `-p <name>` | Specific metadata property |

_Dr. Mārcis Gasūns_
