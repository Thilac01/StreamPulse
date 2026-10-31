# StreamPulse — Codex × Twitch Hackathon

> **STATUS: PLANNING ONLY — DO NOT IMPLEMENT, GENERATE SOURCE CODE, INSTALL DEPENDENCIES, OR DEPLOY UNTIL THE TEAM LEAD EXPLICITLY SAYS `START BUILD`.**

**Project name:** StreamPulse  
**Team:** 6 members, each with one assigned Codex thread and separate Git worktree/branch  
**Initial focus:** Racing-game Twitch streams  
**Submission target:** November 2, 2026 at 2:00 PM Pacific Time (**verify on official event page before submission**)

## 1. What we are building

A contextual audience-engagement companion for Twitch racing streams. It detects **meaningful racing moments**, chooses whether the moment deserves an interaction, asks one rapid prediction (such as “Will the driver complete this overtake?”), reads viewer responses from Twitch chat, displays a live vote split, resolves the outcome, and awards points. **The product also knows when to stay quiet.**

This plan is adapted from the team's *Interactive Audience Engagement for Twitch Streams* concept document. The attached concept document describes the product goal and interactions, **not a verified technical integration**. The racing-first scope and technology choices in this repository are proposed team decisions.

## 2. Why this is different

Ordinary polls ask viewers questions on demand. StreamPulse uses a gated event loop: **racing context → timing score → one prediction → short voting window → verified race outcome → result + lightweight leaderboard**. Prediction questions appear during appropriate windows, not constantly. The streamer has pause/skip controls.

## 3. Primary audiences

- **Streamer:** connects channel, chooses demo racing mode, pauses or skips prompts, watches live results.
- **Viewer:** responds with `!a` / `!b` (or `1` / `2` when a question is active), views the transparent stream overlay, earns session-only points.
- **Demo judge:** sees a reliable end-to-end interaction in less than three minutes.

## 4. MVP boundaries

**Required for demo:** one connected Twitch channel; Twitch OAuth consent; EventSub chat receiving; one contextual racing event; rules-based decision to ask/stay quiet; chat vote counting; one vote per user per question; live overlay; deterministic resolution; score update; streamer start/pause/skip; replayable offline demo fixtures.

**Stretch only if stable:** telemetry ingestion from one supported racing title, AI-assisted wording, OBS browser-source overlay, polished animations, basic streaks.

**Out of scope for hackathon:** generic support for all games, automatic full-screen video understanding, requiring a reviewed Twitch Extension, real-money prediction, payments, extensive player profiling, public production-scale multi-tenant hosting.

## 5. Recommended implementation strategy

**Use a web companion + Twitch chat bot + OBS browser source**, not a reviewed Twitch Extension as the sole MVP. Twitch Extensions have separate developer setup, review, security, and deployment constraints; an independent web overlay lets the team demonstrate without depending on review completion. **Do not represent the OBS overlay as a Twitch-native Extension.**

Initial event ingestion should come from **a controlled racing event simulator or manual trigger**, so the complete loop is reliable. Game telemetry is an optional adapter, not a prerequisite. The scoring logic must never pretend an unverified outcome was detected automatically.

## 6. End-to-end behavior

1. Streamer connects Twitch and selects a demo channel.
2. Race simulator emits `OVERTAKE_OPPORTUNITY` with a unique event ID.
3. Timing policy checks rate limits, quiet periods, duplicates, stream state, and confidence.
4. If eligible, a vote opens for 10–15 seconds: “Will the next overtake succeed?”
5. Viewers vote through chat; bot/EventSub messages are parsed and deduplicated by user and event.
6. Overlay updates live through app WebSocket (or SSE fallback).
7. Event outcome is received from the simulator or validated telemetry source.
8. System reveals the winning answer and adjusts a session leaderboard.
9. Streamer may disable or skip any interaction. When the timing gate rejects a moment, the system stays quiet.

## 7. Three-minute demo outline

| Time | Scene | Proof |
|---|---|---|
| 0:00–0:25 | Streamer dashboard + connected Twitch test channel | Authentic integration |
| 0:25–0:50 | Simulator shows low-value moment; engine stays quiet | Smart timing |
| 0:50–1:15 | Overtake trigger opens context-aware prediction | Relevance |
| 1:15–1:45 | Two or more test viewers vote in Twitch chat | Live chat integration |
| 1:45–2:15 | Poll closes, simulator reports outcome, overlay reveals result | Closed-loop resolution |
| 2:15–2:40 | Leaderboard updates and streamer pauses interactions | Rewards and control |
| 2:40–3:00 | Explain architecture, accessibility, expansion path | Technical credibility |

**Fallback:** recorded game clip + deterministic simulator + actual test-channel chat where possible; disclose which inputs are simulated. Never fake live OAuth or Twitch chat API responses as real.

## 8. Directory map (planned, not created)

```text
README.md
TEAM_CODEX_THREADS.md
technical_architecture.json
apps/
  dashboard/            # streamer UI and overlay frontend
services/
  api/                  # API, orchestration, WebSocket
  twitch-gateway/       # OAuth + EventSub + chat parsing
  event-engine/         # timing policy, question lifecycle, resolution
  game-adapters/        # simulator + optional racing telemetry
packages/
  contracts/            # shared message types and schemas
  ui/                   # optional UI primitives
fixtures/                # synthetic, non-sensitive racing events
scripts/                 # local dev tooling, later
```

## 9. Six-person ownership

| Member | Codex thread | Responsibility |
|---|---|---|
| M1 | `thread-01` | Product owner / integration / demo coordinator |
| M2 | `thread-02` | Twitch OAuth, EventSub and chat pipeline |
| M3 | `thread-03` | Event detection, timing gate and simulator |
| M4 | `thread-04` | Streamer dashboard + transparent overlay |
| M5 | `thread-05` | Vote lifecycle, resolution and scoring API |
| M6 | `thread-06` | Test harness, observability, performance and demo |

Read `TEAM_CODEX_THREADS.md` for non-overlapping file ownership and exact instructions. Technical choices and API contracts are in `technical_architecture.json`.

## 10. Team agreements before coding

- Confirm the selected test channel and whether the bot account is separate; the project name is StreamPulse.
- Get consent from the broadcaster and configure Twitch Developer application and approved redirect URLs.
- Freeze the shared contracts (`question.opened`, `vote.accepted`, `question.closed`, `outcome.resolved`, `leaderboard.updated`, `interaction.skipped`).
- Agree on event timestamps, clock synchronization, valid vote formats and scoring rules.
- Assign one integration owner and one daily integration window.
- Decide whether demo is simulator-only or uses one specific game's permitted telemetry.
- Check competition rules, AI/Codex usage requirements, and submission format independently.

## 11. Security and responsible design

Never commit OAuth access/refresh tokens, Twitch client secrets, broadcaster credentials, or user chats. Use local `.env` / secret manager and a committed `.env.example` **without secrets**. Use least-privilege scopes, OAuth state and PKCE where applicable, server-side token handling, limited retention, chat rate-limit compliance, streamer opt-out, and minimal persistent viewer identifiers. Predictions are entertainment points **without money or prizes of monetary value**.

## 12. Success criteria (proposed targets, not guarantees)

- Full demo completes without manual repair.
- One chat response updates overlay within **1 second p95** under controlled local test conditions (excluding Twitch platform/display latency).
- Vote count is correct, with user-level deduplication; event resolution is idempotent.
- 0 unexpected polls in a quiet-period demonstration.
- No tokens printed in logs, repo, or screenshots.
- Simulator fallback works without game telemetry or AI service.

## 13. Source references

- Original team concept: `Twitch_Codex_Hackathon_Interactive_Audience_Concept.docx` (input document; separately supplied)
- Twitch chat guide: https://dev.twitch.tv/docs/chat/send-receive-messages
- Twitch EventSub WebSockets: https://dev.twitch.tv/docs/eventsub/handling-websocket-events
- Twitch scopes: https://dev.twitch.tv/docs/authentication/scopes/
- Twitch Extensions (optional future route): https://dev.twitch.tv/docs/extensions/
- Codex parallel worktrees: https://learn.chatgpt.com/docs/environments/git-worktrees

---

**Next action:** Agree on this plan. Until the lead explicitly issues `START BUILD`, Codex threads should review requirements and flag conflicts only.
