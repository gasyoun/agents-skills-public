```
ORG=sanskrit-lexicon
REPO=<argument>
```

Verify access and capture domain metadata:
```sh
gh api repos/$ORG/$REPO --jq '{name, description, language, size, has_issues, has_wiki, default_branch}'
```

**Detect repo category** (used in Phase 2 to select label set):

| Heuristic | Category |
|---|---|
| `description` mentions "API", "REST", or PHP backend | `web-backend` |
| `description` mentions "display", "homepage", "app", "frontend" or `language=Dart/JavaScript/HTML` | `web-frontend` |
| `description` mentions "data", "corrections", "source", "sqlite", "json" | `data-store` |
| `description` mentions "convert", "transcoder", "Devanagari", "stardict", "babylon" | `converter` |
| `description` mentions "link", "lslink", "mapping" | `linking-tool` |
| `description` mentions "scan", "scanned edition" | `scanned-book` |
| `description` mentions "headword", "normalization", "inflect", "declension" | `processing-tool` |
| `description` mentions "build", "Hugo", "fonts", "newsletter" | `build-meta` |
| `name` contains `temp_`, `legacy`, or `test_` | `archive` |

Record the detected category. If ambiguous, [ASK] the user once.

---

