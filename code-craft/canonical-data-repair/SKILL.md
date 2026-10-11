---
name: "canonical-data-repair"
description: 'Repair a corrupted canonical data store without fabricating evidence — hash pre-repair snapshot, classify fields RECOVERED/RE-DERIVED/DERIVED/UNRECOVERABLE, dry-run delta, quarantine irrecoverables, verify invariants. Use for corruption/backfill repair.'

---

# canonical-data-repair

Use this skill when the user asks to run the migrated source command `canonical-data-repair`.

## Command Template


# /canonical-data-repair <store> [scope]

_Created: 18-07-2026 · Last updated: 18-07-2026_

Repair canonical data from evidence, never from plausibility. This workflow
mutates the store; `/pipeline-audit` maps causes and consumers, while
`/acceptance-gate` judges whether the repaired pipeline is ready.

## 1. Resolve authority and isolate

Read the repo instructions and cross-repo hubs. Resolve the actual canonical
store, its schema/key invariants, every downstream index, and all surviving
historical artifacts. Refuse an empty, missing, or wrong source root. Work in a
fresh branch/worktree; if another repair lane exists, stop and route the
collision through `/dual-run-salvage`.

Snapshot the pre-repair store before mutation. Record path, row/key count, byte
size, and SHA-256 for the store and every candidate evidence artifact. A file
still being written is not hashable evidence.

## 2. Build the field-level ledger

One ledger row per repaired field:

`record key · field · before · classification · source artifact · source SHA-256 · method · provenance marker · disposition`

Use exactly these classifications:

- `RECOVERED`: copied from a surviving authoritative artifact.
- `RE-DERIVED`: deterministically recomputed from an authoritative artifact;
  record the algorithm/version.
- `DERIVED`: identity/key-derived or otherwise weaker replacement; mark it
  visibly in both ledger and stored provenance immediately.
- `UNRECOVERABLE`: no defensible source; quarantine the record/field.

Every non-`UNRECOVERABLE` row names a non-empty source artifact whose hash
matches the snapshot manifest. No guessed values, silent defaults, or wrong-root
lookups. Identity-derived replacements are permitted only as `DERIVED`, never
mislabelled as recovered.

## 3. Dry-run and publish the exact delta

Run the repair read-only first. Publish exact added/changed/quarantined row and
key sets, field counts by classification, before→after values (redacted where
necessary), source/hash references, and invariant predictions. An unexplained
delta, an empty source, a hash mismatch, or a changed key outside scope is a
hard stop.

Required tabletop cases before write: hashed recovery succeeds; deterministic
re-derivation succeeds; identity derivation is visibly marked; empty/wrong
source is refused; irrecoverable data is quarantined.

## 4. Apply atomically

Re-check source hashes and the canonical preimage hash immediately before the
write. Build the complete replacement in a temporary sibling path, validate it,
then atomically replace the canonical store using the repository's established
mechanism. On mismatch or validation failure, leave the original untouched.

Rebuild all downstream indexes exactly once from the committed canonical state;
never patch indexes independently or rebuild once per row.

## 5. Verify and hand off judgment

Re-read the canonical store and verify schema, unique keys, referential
integrity, counts, null/placeholder policy, quarantine isolation, ledger↔store
agreement, downstream index checksums/counts, and idempotence (a second dry-run
must be empty). Hash the post-repair store, ledger, quarantine, and rebuilt
indexes.

Commit the implementation, manifest, ledger, exact delta, quarantine inventory,
and verification report. Raw sensitive artifacts remain local and are referenced
by safe hashes. Send readiness judgment to `/acceptance-gate`; send architecture
follow-up to `/pipeline-audit`; send any competing-lane comparison to
`/dual-run-salvage`.

## Stop conditions

Stop without mutation if authority is ambiguous, a source/root/hash is invalid,
the preimage changed, the graph has an active competing repair, or atomic
replacement/rollback cannot be guaranteed. Quarantine is a valid repair
outcome; invention is not.

## Compatibility Rule

Write and follow this skill so it works in both Codex and Claude Code. Treat
`$skill-name` references as workflow names: invoke the native skill when
available, otherwise open its canonical runbook. Preserve the evidence and
atomicity contract across hosts.

_Dr. Mārcis Gasūns_
