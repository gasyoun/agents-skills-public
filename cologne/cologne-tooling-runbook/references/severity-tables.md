### Type → default severity

Use as starting point, escalate based on description:

| Type | Default severity |
|---|---|
| `bug` | `minor` (escalate to `major` if user-blocking, `critical` if data-loss) |
| `regression` | `major` (regressions are higher impact than fresh bugs) |
| `security` | `major` minimum, `critical` if exploitable |
| `tech-debt` | `minor` |
| `dependency` | `minor` (`major` for breaking-change upgrades) |
| `feature` | `major` (new capabilities are inherently larger) |
| `enhancement` | `minor` |
| `performance` | `minor` (`major` if user-perceivable lag) |
| `documentation` / `docs-api` | `trivial` (`minor` for full new sections) |
| `infrastructure` | `minor` (`major` if blocks all deploys) |
| `data-pipeline` | `major` (data pipelines touch many records) |
| `cross-repo` | `major` minimum (coordination cost) |
| `build-tooling` | `minor` |
| `question` / `proposal` / `discussion` | `trivial` (research overhead, not implementation) |

---

