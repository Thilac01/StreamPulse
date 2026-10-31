# StreamPulse — Codex × Twitch Hackathon

> **STATUS: PLANNING ONLY.** Do not implement, generate source code, install dependencies, or deploy until the team lead explicitly says `START BUILD`.

- Repository: https://github.com/Thilac01/StreamPulse
- Team: six members, each with one Codex task and a separate branch/worktree
- Initial focus: racing-game Twitch streams
- Product promise: turn meaningful racing moments into timely, lightweight viewer predictions

## What we are building

StreamPulse is a contextual audience-engagement companion for Twitch racing streams. It receives a meaningful racing event, decides whether that moment deserves an interaction, opens one rapid prediction, reads viewer votes from Twitch chat, shows the live split, resolves the outcome, and awards session points. The product must also know when to stay quiet.

The reliable MVP uses a controlled racing-event simulator or manual trigger. A permitted game-telemetry adapter is a stretch goal. The system must disclose simulated inputs and must never claim an outcome was automatically detected when it was not.

## MVP boundaries

Required for the demo:

- One connected Twitch test channel with broadcaster consent
- Twitch OAuth consent and EventSub chat receiving
- One contextual racing event, initially `OVERTAKE_OPPORTUNITY`
- Rules-based timing decision: ask or stay quiet
- One vote per Twitch user per question
- Live transparent web overlay
- Deterministic outcome resolution and session-only scoring
- Streamer start, pause, resume, and skip controls
- Replayable offline fixtures for a reliable fallback

Stretch only after the complete loop is stable:

- One supported game's permitted telemetry
- AI-assisted question wording with deterministic fallback
- OBS browser-source polish and animations
- Lightweight streaks

Out of scope:

- Generic support for every game
- Full-screen video understanding
- Depending on Twitch Extension review for the MVP
- Money, prizes of monetary value, or wagering
- Public production-scale multi-tenancy
- Extensive viewer profiling

## End-to-end behavior

1. The streamer connects Twitch and selects the consented demo channel.
2. The simulator emits an event with a unique `eventId`.
3. The timing policy checks state, confidence, cooldown, quiet period, and duplicates.
4. If eligible, a 10–15 second prediction opens.
5. Viewers vote with `!a`, `!b`, `1`, or `2` while the question is active.
6. The backend sends live counts to the overlay by WebSocket, with SSE as a fallback.
7. The simulator or validated telemetry source reports the outcome.
8. Resolution is idempotent; the result is revealed and the session leaderboard updates.
9. The streamer can pause or skip. Rejected moments produce no viewer interruption.

## Three-minute demo

| Time | Scene | Proof |
|---|---|---|
| 0:00–0:25 | Dashboard and connected test channel | Authentic Twitch integration |
| 0:25–0:50 | Low-value simulated moment | Timing engine stays quiet |
| 0:50–1:15 | Overtake opportunity | Context-aware prediction opens |
| 1:15–1:45 | Test viewers vote in Twitch chat | Live ingestion and deduplication |
| 1:45–2:15 | Outcome arrives and result appears | Closed-loop resolution |
| 2:15–2:40 | Leaderboard updates; streamer pauses | Rewards and streamer control |
| 2:40–3:00 | Architecture and fallback summary | Technical credibility |

Fallback: use a recorded racing clip, deterministic simulator, and real test-channel chat where possible. Clearly disclose which inputs are simulated.

## Planning pack

- `AGENTS.md` — repository-wide Codex rules and build authorization
- `TEAM_CODEX_THREADS.md` — six non-overlapping roles and task prompts
- `technical_architecture.json` — machine-readable architecture and event catalog
- `docs/implementation_plan.md` — phases, dependencies, tests, risks, and acceptance criteria
- `docs/integration_contracts.md` — APIs, events, data schemas, and state rules

## Planned repository map

```text
StreamPulse/
├── README.md
├── AGENTS.md
├── TEAM_CODEX_THREADS.md
├── technical_architecture.json
├── docs/
│   ├── implementation_plan.md
│   └── integration_contracts.md
├── apps/dashboard/          # streamer UI and transparent overlay
├── services/api/            # orchestration and realtime transport
├── services/twitch-gateway/ # OAuth, EventSub, chat parsing
├── services/event-engine/   # timing, question lifecycle, resolution
├── services/game-adapters/  # simulator and optional telemetry
├── packages/contracts/      # shared types and schemas
└── fixtures/                # synthetic, non-sensitive events
```

## Success criteria

- The full demo completes without manual repair.
- A valid chat response updates the overlay within 1 second p95 in controlled local testing, excluding Twitch platform/display latency.
- Vote counts are correct and deduplicated per question and viewer.
- Outcome resolution and scoring are idempotent.
- The quiet-period demonstration opens zero unexpected questions.
- No token, client secret, credential, or raw chat log appears in commits, logs, or screenshots.
- The simulator fallback works without game telemetry or an AI service.

## Before coding

The team lead must freeze the contracts, select the test channel and bot identity, configure a Twitch developer application and redirect URLs, choose simulator-only versus a specific permitted telemetry source, and verify current competition rules and submission details.

The only build authorization phrase is `START BUILD`. Until it is issued, Codex tasks may review and improve planning documents but must not create application code or install dependencies.

