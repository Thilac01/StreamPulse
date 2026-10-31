# StreamPulse Integration Contracts — v1 Planning Draft

## Contract conventions

- JSON over HTTPS for commands/snapshots; WebSocket for realtime events, with SSE fallback if selected.
- UTC ISO 8601 timestamps, opaque string IDs, UTF-8, camelCase fields.
- Every event envelope includes `schemaVersion`, `type`, `eventId`, `occurredAt`, `correlationId`, and `data`.
- Delivery is at least once. Consumers deduplicate by `eventId`; commands use `Idempotency-Key`.
- The API/event engine is authoritative for lifecycle state, eligibility, totals, resolution, and scores.
- Unsupported major schema versions fail clearly; additive optional fields are tolerated within v1.

## Event envelope

```json
{
  "schemaVersion": "1.0",
  "type": "question.opened",
  "eventId": "evt_01",
  "occurredAt": "2026-10-31T10:00:00.000Z",
  "correlationId": "race_01",
  "data": {}
}
```

## Core events

### `racing.event.detected`

Producer: M3. Consumer: M5.

Required `data`: `raceEventId`, `kind`, `source`, `sourceEventId`, `confidence`, `detectedAt`, `participants`, and optional `context`. Initial `kind` is `OVERTAKE_OPPORTUNITY`; `source` is `SIMULATOR` or `TELEMETRY`.

### `interaction.skipped`

Producer: timing policy. Consumers: dashboard and observability.

Required `data`: `raceEventId`, `reason`. Reasons: `PAUSED`, `LOW_CONFIDENCE`, `COOLDOWN`, `QUIET_PERIOD`, `DUPLICATE`, `QUESTION_ACTIVE`, or `STREAM_NOT_READY`.

### `question.opened`

Required `data`: `questionId`, `raceEventId`, `prompt`, `options`, `openedAt`, `closesAt`, `status`, and `revision`. Options have stable IDs `A` and `B` and labels. `status` is `OPEN`.

### `chat.message.received`

Producer: M2. Consumer: M5. This event should be short-lived and not persisted as a raw chat archive.

Required `data`: `messageId`, `channelId`, `viewerKey`, `receivedAt`, and `normalizedCommand`. `normalizedCommand` is `A`, `B`, or `NONE`. `viewerKey` is a session-scoped/pseudonymous identifier where feasible.

### `vote.accepted` and `vote.rejected`

Accepted data: `questionId`, `voteId`, `viewerKey`, `optionId`, `acceptedAt`, `totals`, `revision`.

Rejected data: `questionId` if known, `messageId`, `reason`, `rejectedAt`. Reasons: `NO_ACTIVE_QUESTION`, `DUPLICATE_VIEWER`, `INVALID_OPTION`, `LATE`, `CHANNEL_MISMATCH`, or `PAUSED`.

### `question.closed`

Required `data`: `questionId`, `closedAt`, `closeReason`, `totals`, and `revision`. Reasons: `TIMER`, `STREAMER_SKIP`, `PAUSE`, or `SYSTEM_ERROR`. Only `TIMER` proceeds to normal scoring.

### `racing.outcome.detected`

Producer: M3. Consumer: M5.

Required `data`: `raceEventId`, `sourceOutcomeId`, `outcome`, `confidence`, and `detectedAt`. Initial outcomes: `SUCCESS`, `FAILURE`, `VOID`. Outcomes must be traceable to a simulator fixture or validated telemetry source.

### `outcome.resolved`

Required `data`: `questionId`, `raceEventId`, `result`, `winningOptionId`, `resolvedAt`, `totals`, and `revision`. `winningOptionId` is null for `VOID`. Repeated equivalent outcome commands return the original resolution; conflicting outcomes fail and alert.

### `leaderboard.updated`

Required `data`: `sessionId`, `questionId`, `entries`, `updatedAt`, and `revision`. Each entry contains `viewerKey`, `points`, and `rank`. Do not include display names unless separately consented and required for the demo.

## Question state machine

```text
IDLE -> OPEN -> CLOSED -> RESOLVED
          |        |
          +------> SKIPPED
```

- Only one question may be `OPEN` per channel/session.
- Votes are accepted only in `OPEN` and when `receivedAt < closesAt`.
- `pause` prevents new questions. Policy choice: it closes an active question as `PAUSE` without scoring.
- `skip` transitions an active question to `SKIPPED` without scoring.
- Resolution requires a timer-closed question and matching `raceEventId`.
- Every state change increments `revision`; clients ignore older revisions.

## Planned HTTP API

| Method and path | Purpose | Key behavior |
|---|---|---|
| `GET /api/v1/session` | Authoritative dashboard snapshot | No secrets; includes connection and lifecycle state |
| `POST /api/v1/session/start` | Start demo session | Idempotent; requires streamer authorization |
| `POST /api/v1/session/pause` | Pause interactions | Prevents new questions; closes active as unscored |
| `POST /api/v1/session/resume` | Resume interactions | Does not reopen previous question |
| `POST /api/v1/questions/{id}/skip` | Skip active question | Idempotent; no scoring |
| `POST /api/v1/simulator/events` | Submit approved fixture event | Demo/dev only; disabled in production mode |
| `POST /api/v1/simulator/outcomes` | Submit approved fixture outcome | Demo/dev only; idempotent |
| `GET /api/v1/overlay/snapshot` | Overlay recovery snapshot | Minimal public data; short-lived overlay access token if needed |
| `GET /api/v1/realtime` | WebSocket/SSE negotiation | Sends snapshot then ordered revisions |

## Error shape

```json
{
  "error": {
    "code": "QUESTION_NOT_OPEN",
    "message": "The question is not accepting votes.",
    "correlationId": "race_01",
    "retryable": false
  }
}
```

Never return tokens, provider payloads containing sensitive data, or internal stack traces.

## Scoring proposal

- Correct answer: 100 session points.
- Incorrect, duplicate, late, skipped, or void question: 0 points.
- A `(questionId, viewerKey)` pair can receive points once.
- A question can update the leaderboard once, guarded by the resolution idempotency record.
- Scores reset with the demo session and are not a wallet or transferable value.

## Contract validation cases

- Duplicate `racing.event.detected` opens at most one question.
- Duplicate chat message and two different messages from the same viewer count once.
- A message at or after `closesAt` is rejected as late using server receipt time.
- Duplicate matching outcome returns the original result without double-scoring.
- Conflicting second outcome is rejected and creates an operator alert.
- Realtime reconnect begins with an authoritative snapshot and then newer revisions.
- A paused or skipped question cannot award points.
- Logs carry `correlationId` but omit secrets and raw message content.

