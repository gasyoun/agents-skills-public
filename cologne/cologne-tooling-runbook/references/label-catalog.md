### Type labels (color `0e8a16` for code-quality, `0075ca` for features, `d4c5f9` for docs, `f9d0c4` for infra, `e99695` for research)

```sh
# Code Quality & Maintenance — green 0e8a16
for label in bug regression tech-debt dependency security; do
  gh api repos/$ORG/$REPO/labels -X POST -f name="$label" -f color="0e8a16" 2>/dev/null || true
  gh api repos/$ORG/$REPO/labels/$label -X PATCH -f color="0e8a16" 2>/dev/null || true
done

# Features & Enhancements — blue 0075ca
for label in feature enhancement performance; do
  gh api repos/$ORG/$REPO/labels -X POST -f name="$label" -f color="0075ca" 2>/dev/null || true
  gh api repos/$ORG/$REPO/labels/$label -X PATCH -f color="0075ca" 2>/dev/null || true
done

# Documentation — purple d4c5f9
for label in documentation docs-api; do
  gh api repos/$ORG/$REPO/labels -X POST -f name="$label" -f color="d4c5f9" 2>/dev/null || true
done

# Infrastructure & Operations — peach f9d0c4
for label in infrastructure data-pipeline cross-repo build-tooling; do
  gh api repos/$ORG/$REPO/labels -X POST -f name="$label" -f color="f9d0c4" 2>/dev/null || true
done

# Research & Discussion — pink e99695
for label in question proposal discussion; do
  gh api repos/$ORG/$REPO/labels -X POST -f name="$label" -f color="e99695" 2>/dev/null || true
done
```

### Severity labels (4 levels — one extra over dictionary repos)

```sh
gh api repos/$ORG/$REPO/labels -X POST -f name="trivial"  -f color="c2e0c6" 2>/dev/null || true
gh api repos/$ORG/$REPO/labels -X POST -f name="minor"    -f color="e4e669" 2>/dev/null || true
gh api repos/$ORG/$REPO/labels -X POST -f name="major"    -f color="fbca04" 2>/dev/null || true
gh api repos/$ORG/$REPO/labels -X POST -f name="critical" -f color="b60205" 2>/dev/null || true
```

### Domain labels (color `5319e7` — purple, scoped per repo category)

Apply the subset matching the repo's Phase 0 category:

| Repo category | Domain labels to create |
|---|---|
| `web-backend` | `domain:api`, `domain:auth`, `domain:db`, `domain:caching` |
| `web-frontend` | `domain:ui`, `domain:routing`, `domain:i18n`, `domain:rendering` |
| `data-store` | `domain:schema`, `domain:migration`, `domain:integrity`, `domain:storage` |
| `converter` | `domain:transcoding`, `domain:roundtrip`, `domain:edge-cases` |
| `linking-tool` | `domain:link-resolution`, `domain:source-mapping`, `domain:coverage` |
| `processing-tool` | `domain:morphology`, `domain:normalization`, `domain:lookup` |
| `scanned-book` | `domain:ocr`, `domain:image-quality`, `domain:metadata` |
| `build-meta` | `domain:ci`, `domain:packaging`, `domain:publishing` |
| `archive` | (skip domain labels — preserve as-is) |

```sh
# Example for web-backend
for label in api auth db caching; do
  gh api repos/$ORG/$REPO/labels -X POST -f name="domain:$label" -f color="5319e7" 2>/dev/null || true
done
```

---

