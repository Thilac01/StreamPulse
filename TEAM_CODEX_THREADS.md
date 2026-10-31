# StreamPulse — Six Codex Roles

> Planning only until the team lead issues `START BUILD`.

Each member should open one Codex task in a separate Git worktree/branch, paste the common prompt below, then paste only their role prompt. Role boundaries are designed to reduce merge conflicts.

## Common prompt for every task

```text
You are working on StreamPulse, a racing-first Twitch audience-engagement MVP.
Read README.md, AGENTS.md, TEAM_CODEX_THREADS.md, technical_architecture.json,
docs/implementation_plan.md, and docs/integration_contracts.md.

Current mode is PLAN ONLY. Do not generate application code, initialize a project,
install dependencies, deploy, or access production systems unless the team lead has
written the exact phrase START BUILD in this task.

Stay within your ownership boundaries. Shared-contract changes must be proposed to
Member 1. Return: Goal, Files affected, Proposed approach, Dependencies, Tests and
validation, Risks, and Status.
```

## M1 — Product, architecture, and integration owner

- Branch: `m1/integration`
- Owns: root planning files, `packages/contracts/**`, integration decisions, demo script
- Avoids: implementing other services except minimal integration glue after authorization

```text
Act as Member 1, the product and integration owner. Reconcile requirements, freeze
event/API contracts, maintain acceptance criteria, sequence integration, and resolve
cross-role conflicts. Create an integration checklist and identify decisions still
needed from the team lead. Do not build yet.
```

Deliverables: frozen contract proposal, dependency map, decision log, integration order, and demo go/no-go checklist.

## M2 — Twitch identity, EventSub, and chat gateway

- Branch: `m2/twitch`
- Owns: `services/twitch-gateway/**`
- Reads: shared contracts; proposes changes to M1

```text
Act as Member 2. Plan Twitch OAuth, broadcaster/channel selection, EventSub WebSocket
session handling, chat-message ingestion, reconnect/retry behavior, vote-command
normalization, scope minimization, token storage, and rate-limit handling. Verify
implementation assumptions against current official Twitch documentation when build
work begins. Define test doubles and failure cases. Do not build yet.
```

Deliverables: OAuth flow, EventSub lifecycle, normalized chat-message contract, Twitch test plan, and secrets checklist.

## M3 — Racing events, simulator, and timing policy

- Branch: `m3/events`
- Owns: `services/game-adapters/**`, timing policy portion of `services/event-engine/**`, `fixtures/racing/**`

```text
Act as Member 3. Plan the deterministic racing simulator, event normalization, event
confidence, deduplication, cooldown and quiet-period policy, ASK/SKIP decision reasons,
and optional telemetry adapter boundary. Define a replayable scenario containing one
rejected low-value event and one accepted overtake opportunity. Do not build yet.
```

Deliverables: simulator scenario, normalized racing-event schema proposal, timing decision table, and deterministic tests.

## M4 — Streamer dashboard and viewer overlay

- Branch: `m4/frontend`
- Owns: `apps/dashboard/**`, `packages/ui/**`

```text
Act as Member 4. Plan the streamer dashboard and transparent browser-source overlay.
Cover disconnected, connecting, ready, paused, question-open, resolved, skipped, and
error states. Include keyboard operation, color-independent vote display, readable
type at stream resolution, reconnect behavior, and a reduced-motion mode. Do not build.
```

Deliverables: screen/state inventory, realtime subscription plan, accessibility checklist, overlay acceptance tests, and visual fallback behavior.

## M5 — Question lifecycle, voting, resolution, and scoring

- Branch: `m5/voting`
- Owns: voting/lifecycle portion of `services/event-engine/**`, relevant `services/api/**`

```text
Act as Member 5. Plan the authoritative question state machine, vote validation and
per-user deduplication, close timing, idempotent outcome resolution, session scoring,
leaderboard updates, and control APIs. Specify concurrency and retry behavior. Use
only entertainment points. Do not build yet.
```

Deliverables: state machine, endpoint behavior, idempotency rules, score rules, and lifecycle test matrix.

## M6 — Quality, observability, performance, and demo reliability

- Branch: `m6/quality`
- Owns: `tests/**`, `scripts/**`, non-sensitive demo fixtures outside M3's racing fixtures, quality documentation

```text
Act as Member 6. Plan the end-to-end test harness, contract tests, load/latency checks,
structured logging and metrics, secret scanning, failure injection, offline fallback,
and three-minute demo rehearsal. Define pass/fail evidence for every acceptance
criterion. Do not build yet.
```

Deliverables: test pyramid, trace/metric catalog, latency method, rehearsal checklist, fallback drill, and go/no-go report template.

## Integration order after authorization

1. M1 freezes v1 contracts and publishes fixtures.
2. M3 produces deterministic event and outcome fixtures; M5 validates the lifecycle against them.
3. M4 renders contract fixtures before live transport is connected.
4. M2 feeds normalized chat messages into M5's vote boundary.
5. M1 connects the complete vertical slice.
6. M6 runs contract, failure, latency, security, and full-demo checks.

## Handoff rule

A handoff must name the producing role, consuming role, contract and version, example fixture, validation evidence, known limitations, and owner of the next action. No role should merge a breaking shared-contract change without M1 approval and acknowledgements from affected consumers.

