**6c. Cost & run-notes — always document the job's cost to an md.** Inside the root-README `## Front matter` section, add a **collapsible run-notes block** so every run leaves a cost trail:

```
<details>
<summary><strong>OCR run notes (YYYY-MM-DD)</strong> — cost, timing, and technical lessons</summary>

Produced by the `/sanscritica-ocr` skill (vision OCR + translation subagents). Process retrospective, not part of the deliverable.

**Cost.** Subagents (exact, from harness telemetry): N agents, ≈XXX,XXX output tokens, NN tool calls — <OCR agents …> + <translation agents …>. Main thread (estimate, dominated by ~NN native-resolution crop reads + any hand repairs): ≈NNNk tokens. **Total ≈X.X M tokens.**

**Time.** Wall-clock ≈NN min. <which waves overlapped and what gated each>.

**Technical lessons (reusable):** <numbered list of what bit you this run>.

</details>
```

Tally the cost from harness telemetry: every subagent result carries a `<usage>` block with `subagent_tokens`, `tool_uses`, `duration_ms`. Sum the OCR agents and the translation agents **separately**, and note any agent that died on a 400/500 error — it reports **0 tokens** but still burned tool calls. Add a main-thread estimate (the foreground native-resolution crop→read loop and any hand repairs dominate it), then give a grand total in tokens. Wall-clock: fan-out waves overlap, so each wave is gated by its slowest agent — report that, not the sum. This block lives in the `<details>` so it documents process without bloating the deliverable. If a repo prefers a standalone file, write `<repo>/prefaces/RUN_NOTES.md` instead and link it from the root README.
