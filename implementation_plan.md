# StreamPulse Implementation Plan

## Status and objective

This is a planning document. Implementation is unauthorized until the team lead writes `START BUILD`.

The objective is a dependable racing-first Twitch MVP that demonstrates one complete, honest interaction loop in under three minutes. The deterministic simulator is the primary event source; live game telemetry and AI-generated wording are optional.

## Decisions required before build

| Decision | Owner | Blocks |
|---|---|---|
| Select consented Twitch test channel and bot identity | Team lead + M2 | Live OAuth/chat test |
| Register Twitch app and approve exact local/demo redirect URLs | Team lead + M2 | OAuth |
| Confirm simulator-only or name one permitted telemetry source | Team lead + M3 | Adapter scope |
| Freeze question duration and score values | M1 + M5 | Lifecycle contracts |
| Choose WebSocket primary and SSE fallback behavior | M1 + M4 + M5 | Realtime contract |
| Verify current event rules, AI policy, submission deadline, and format | Team lead + M1 | Submission |

## Phase plan

### Phase 0 — Freeze contracts and evidence

Owners: M1, with all members reviewing.

- Finalize schema version `1.0`, API error shape, IDs, timestamps, and state names.
- Publish valid and invalid fixtures for every cross-service event.
- Record security assumptions and the source of each Twitch integration requirement.
- Map every demo scene to observable evidence and a fallback.

Exit: all six roles acknowledge the v1 contracts; unresolved items are documented and owned.

### Phase 1 — Deterministic vertical slice

Owners: M3, M5, and M4; M1 integrates.

- Replay a low-value event that the timing policy skips.
- Replay one `OVERTAKE_OPPORTUNITY` that opens a question.
- Accept fixture votes, close the question, resolve the outcome once, and update scores.
- Render the complete lifecycle in dashboard and overlay states.

Exit: end-to-end flow works locally without Twitch, game telemetry, or AI.

### Phase 2 — Twitch integration

Owner: M2; M5 consumes normalized chat messages.

- Complete server-side OAuth flow using least privilege.
- Establish EventSub session, subscribe, receive messages, and recover from reconnect.
- Normalize active-window vote commands and discard irrelevant chat.
- Demonstrate token redaction and controlled retry behavior.

Exit: two test viewers can vote in the consented channel; duplicates and late votes are rejected correctly.

### Phase 3 — Realtime controls and resilience

Owners: M4 and M5; M6 validates.

- Connect dashboard controls and overlay transport.
- Add snapshot-on-reconnect and monotonic revision handling.
- Exercise pause, resume, skip, duplicate delivery, transport loss, and delayed outcome.
- Confirm deterministic recovery or a clear operator error state.

Exit: temporary client disconnects do not corrupt authoritative state or scores.

### Phase 4 — Demo hardening

Owner: M6; all roles participate.

- Run the three-minute rehearsal repeatedly from a clean start.
- Measure vote-to-overlay latency and record test conditions.
- Run secret scanning, contract tests, accessibility checks, and fallback drill.
- Freeze stable scope. Stretch features require unanimous go/no-go approval.

Exit: three consecutive successful rehearsals plus one successful offline-fallback rehearsal.

## Dependency map

| Producer | Contract | Consumers |
|---|---|---|
| M3 | `racing.event.detected`, `racing.outcome.detected` | M5 timing/lifecycle |
| M2 | `chat.message.received` | M5 voting |
| M5 | question, vote, result, leaderboard events; control APIs | M4 UI, M6 tests |
| M1 | shared schemas and fixtures | All roles |
| M4 | visible states and operator feedback | M6 demo tests |
| M6 | validation evidence and go/no-go | M1/team lead |

## Test strategy

| Layer | Required checks |
|---|---|
| Schema | Valid fixtures accepted; missing fields, unsupported versions, and invalid enums rejected |
| Unit | Timing decision table, vote parsing, deduplication, scoring, state transitions |
| Component | OAuth state/PKCE handling, EventSub reconnect, question timers, realtime snapshots |
| Contract | Every producer fixture consumed by the owning downstream component |
| End-to-end | Quiet event; accepted event; two viewers; duplicate/late vote; resolution; leaderboard; pause |
| Failure | Twitch disconnect, duplicate event, overlay reconnect, delayed/duplicate outcome, service restart |
| Security | Secret scanning, log redaction, redirect validation, least scopes, control authorization |
| Accessibility | Keyboard controls, focus visibility, color-independent meaning, contrast, reduced motion |
| Performance | Controlled p50/p95 vote-to-overlay latency; event burst does not open overlapping questions |

## Demo acceptance criteria

- The dashboard identifies the configured test channel without exposing credentials.
- A low-value racing moment produces a visible operator reason for `SKIP` and no viewer question.
- One overtake event opens exactly one question for 10–15 seconds.
- At least two test viewers can vote from Twitch chat.
- A second vote by the same viewer does not increase totals.
- The overlay updates within the stated controlled-test latency target.
- A deterministic outcome closes and resolves the question exactly once.
- Correct voters receive session points and the leaderboard updates once.
- Pause prevents a new question; skip closes an active question without scoring.
- The offline simulator fallback completes without Twitch/game telemetry/AI and is identified as simulated.
- Logs, screenshots, fixtures, and commits contain no secrets or raw chat export.

## Principal risks

| Risk | Mitigation | Owner |
|---|---|---|
| Twitch setup or policy mismatch | Verify current official docs early; keep offline path | M2 |
| Contract drift across six branches | M1 owns schemas; fixture-based contract tests | M1/M6 |
| Race between close, vote, and outcome | Server timestamps, state machine, idempotency keys | M5 |
| Too many prompts | Cooldown, quiet period, confidence threshold, one-active-question rule | M3/M5 |
| Demo network failure | Deterministic simulator, cached fixtures, clear disclosure | M6 |
| Stretch work destabilizes MVP | Freeze core after Phase 3; explicit go/no-go | M1 |
| Sensitive Twitch data leaks | Least privilege, redaction, no real chat fixtures | M2/M6 |

## Build authorization checklist

Before `START BUILD`, M1 should confirm that required decisions are closed, all six role owners accept their boundaries, the v1 contracts are frozen, the team has broadcaster consent, and the current competition requirements have been independently verified.

