---
name: "sli-probe-build"
description: 'Build ONE SLI/probe from definition to live receipt: define the measurable fact → implement read-only on an EXTERNAL runner (never probe prod from itself) → wire (Better Stack / launchd / cron) → prove firing with a real receipt → register in the SLO kit + dated GTD loop. Triggers: "build an SLI", "add a probe", "guard pack", "journey probe", "is this measurable". Mechanical — OxAlpha tier; the SLI definition is judgment.'

---

# sli-probe-build

Use this skill when the user asks to run the migrated source command `sli-probe-build`.

## Command Template


# /sli-probe-build <what-to-measure>

_Created: 07-09-2026 · Last updated: 07-09-2026_

Distills the thirteen-session SRE wave of 04–07-09-2026 that kept re-deriving the same ladder: the agent-fleet SLO monthly review ([H4152](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H4152-OxAlpha_Uprava_agent-fleet-slo-review-0410_05.09.26.md)), product journey probes + SLI compute ([H4168](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H4168-OxAlpha_Uprava_prod-journey-probes-sli-compute_05.09.26.md)), multiwindow burn-rate alerting ([H4169](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H4169-OxAlpha_Uprava_burn-rate-alerting-product-budget_05.09.26.md)), the off-site restore drill + durability SLI ([H4170](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H4170-OxAlpha_Uprava_offsite-restore-drill-durability-sli_05.09.26.md)), the external-dep SLI + fallback registry ([H4171](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H4171-OxAlpha_Uprava_external-dep-sli-fallback-registry_05.09.26.md)), the .92 guard pack ([H4194](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H4194-OxAlpha_Uprava_systema92-guard-pack-resources-queue-compromise_06.09.26.md)), journey-probe extensions ([H4196](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H4196-OxAlpha_Uprava_journey-probe-tls-expiry-calendar-deadman_06.09.26.md)), the <client> liveness probe ([H4197](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H4197-OxAlpha_Uprava_<client>-site-liveness-probe_06.09.26.md)), the Mac probe-copy refresh + launchd restamp ([H4198](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H4198-OxAlpha_Uprava_probe-journeys-mac-deploy-refresh_06.09.26.md)), SRE cadence recurrence rows ([H4244](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H4244-OxAlpha_Uprava_sre-cadence-recurrence-rows_06.09.26.md)), the external-runner null-route lesson ([H4287](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H4287-OxAlpha_Uprava_hourly-journey-probes-external-runner_07.09.26.md)), pipeline correctness SLIs ([H4288](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H4288-OxAlpha_Uprava_pipeline-correctness-active-lanes_07.09.26.md)), and the response-time SLI ([H4292](https://github.com/gasyoun/Uprava/blob/main/handoffs/archive/H4292-OxAlpha_Uprava_response-time-sli-timestamp-source_07.09.26.md)). Approved **C46** in the skill-mine delta census ([SKILL_CANDIDATES_CENSUS_H4060_H4304_07_09_2026.md](https://github.com/gasyoun/Uprava/blob/main/SKILL_CANDIDATES_CENSUS_H4060_H4304_07_09_2026.md)), MG chat vote 5/5 approve 07-09-2026.

The discipline this owns: **a measurement that never fired is folklore.** One run of this skill takes an SLI from a chat sentence to a wired, registered, receipt-bearing loop.

## Phase 0 — Define the measurable fact, locate the honest source

- One sentence: what number, over what population, what is "red". If you cannot name the red condition, you have a vibe, not an SLI.
- **Locate/verify the source BEFORE building** (H4292): does the action-timestamp/dataset the SLI needs actually exist, where, at what grain? An SLI whose source is a daily-grain TSV cannot feed an intraday page (H4245 lesson) — say so and degrade the tier honestly.

## Phase 1 — Implement read-only, external runner

- The probe never writes to the system it measures. **Never probe prod from prod itself** — the prober runs on a different box/runner (H4287 null-route: a probe that an attacker controls or that dies with the host proves nothing).
- Offline-first: the probe works without secrets in env where possible; credentials come from the box's own store, never hardcoded.

## Phase 2 — Wire into the reminder path

- Better Stack heartbeat/cron/launchd per the surface: scheduled = plist or cron with `SOURCE_COMMIT` restamp on deploy (H4198), alerting = heartbeat + burn-rate windows (H4169).
- Every tier is a TICKET WITH A DEADLINE (H4245 kit amendment): page-equivalent → action by 10:00 next morning; never a 3 a.m. ring for a batch-deciding human unless the page tier is actually armed.

## Phase 3 — Prove it fires (receipt, not hope)

- A probe that has never fired is UNVERIFIED. Force one real firing through the wired path (test alert, deliberate threshold crossing on a test SKU) and keep the receipt — HTTP 200, incident id, heartbeat line (H4245: `/fail` ping → incident dispatched to the phone).
- Record expected-reason for any liveness RED (H4179 lesson): a red without a reason is a defect to diagnose, not noise to mute.

## Phase 4 — Register + cadence

- Fleet/SLO-kit table row: name, target, window, owner, runner, last_verified.
- Dated GTD loop for the recurring read (monthly SLO review, weekly recheck) — self-perpetuating convention (H4244: five dated loop rows), never "someone should look at this".

## Dedupe fence

- [`/source-health-probe`](https://github.com/gasyoun/claude-config/blob/main/commands/source-health-probe.md) — Sanskrit external sources only, outage log, no alerting ladder.
- [`/sos`](https://github.com/gasyoun/claude-config/blob/main/commands/sos.md) — live incident response, not measurement.
- [`/acceptance-gate`](https://github.com/gasyoun/claude-config/blob/main/commands/acceptance-gate.md) — readiness packet for a pipeline, not a standing SLI.
- [`/n8n`](https://github.com/gasyoun/claude-config/blob/main/commands/n8n.md) — host-ops units on .91; an SLI may ride its box but is not a host-ops unit.
- [`/ssh`](https://github.com/gasyoun/claude-config/blob/main/commands/ssh.md) — transport to .92; this skill owns what runs there and what proves it works.
