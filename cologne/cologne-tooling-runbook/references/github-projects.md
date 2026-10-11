
Tool repos use **two cross-repo org projects** (separate from the four dictionary projects):

- **Tooling: Stability & Quality** — for `bug`, `regression`, `security`, `dependency`, `performance` work
- **Tooling: Capabilities & Roadmap** — for `feature`, `enhancement`, `proposal`, `cross-repo` work

Discover or create:
```sh
gh api graphql -f query='{ organization(login:"sanskrit-lexicon") {
  projectsV2(first:30) { nodes { id number title } } } }'
```

If the two tooling projects don't exist, [ASK] once before creating org-level projects.

Mapping:
| Project | Types |
|---|---|
| Tooling: Stability & Quality | `bug`, `regression`, `security`, `dependency`, `performance`, `data-pipeline` |
| Tooling: Capabilities & Roadmap | `feature`, `enhancement`, `proposal`, `cross-repo`, `discussion` |
| (no project) | `tech-debt`, `documentation`, `docs-api`, `infrastructure`, `build-tooling`, `question` |

Assign via GraphQL `addProjectV2ItemById` (same pattern as dictionary runbook Phase 6).

---

