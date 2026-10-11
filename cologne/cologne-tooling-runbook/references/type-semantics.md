### Type semantics

| Label | Apply when |
|---|---|
| `bug` | Code defect — produces wrong output, throws unexpectedly, violates a documented contract |
| `regression` | Previously working behavior, now broken (mention which version/commit if known) |
| `tech-debt` | Refactoring, dead-code removal, modernization, type-hint additions, consistency cleanup |
| `dependency` | Library upgrade, vendor swap, security patch (CVE), runtime version bump |
| `security` | CVE, credential exposure, injection vector, auth bypass — escalates severity to `major` minimum |
| `feature` | Net-new capability that didn't exist before |
| `enhancement` | Improvement to an existing capability (faster, clearer, more flexible) — NOT new |
| `performance` | Speed, memory, throughput optimization without changing behavior |
| `documentation` | Prose docs: README, CONTRIBUTING, code comments, examples |
| `docs-api` | API contract / schema / endpoint documentation specifically (separate so backend teams can filter) |
| `infrastructure` | CI/CD, deploy scripts, Docker, GitHub Actions, environment setup |
| `data-pipeline` | ETL, batch jobs, scheduled processing, cron scripts that move/transform data |
| `cross-repo` | Issue requires changes in 2+ repos — link to companion issues in body |
| `build-tooling` | Build scripts, package generation, release tooling |
| `question` | Needs research/clarification before any code change |
| `proposal` | Architecture/design proposal — use for RFC-style discussions |
| `discussion` | Open-ended; not yet a proposal |

### Disambiguation rules

- "Add Russian translation" → `feature` (new capability), not `enhancement`
- "Russian translation has typo" → `bug`
- "Russian translation flow is slow" → `performance`
- "Russian translation needs better docs" → `documentation`
- "Should we add Russian translation?" → `proposal` (decision needed first)
- "How do other dictionaries handle Russian?" → `question` (research needed first)

---

