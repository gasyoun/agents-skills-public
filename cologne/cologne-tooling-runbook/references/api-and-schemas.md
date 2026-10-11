
Add `## API` or `## Data Schema` to CLAUDE.md (depending on repo category):

For `web-backend`:
```markdown
## API Endpoints

| Method | Path | Purpose |
|---|---|---|
| GET | /v1/<resource> | <description> |
```

For `data-store` / `converter`:
```markdown
## Data Schema

Input: <format, encoding, validation rules>
Output: <format, encoding, guarantees>
Invariants: <what must always hold>
```

For `linking-tool`:
```markdown
## Mapping Schema

Source identifier format: <example>
Target identifier format: <example>
Resolution algorithm: <one-paragraph summary>
```

---

## Phase 14 — Encoding & locale declaration

Tool-repo specific encoding statement:
```markdown
## Encoding

- **File encoding**: UTF-8 NFC
- **Sanskrit text**: SLP1 in code; IAST/Devanagari only at display layer (via transcoder)
- **Locale**: <en_US.UTF-8 default; ru_RU.UTF-8 for newsletter; etc.>
- **Line endings**: LF (enforced via `.gitattributes`)
```

---

