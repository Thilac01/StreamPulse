# StreamPulse — First Codex prompts (planning only)

## Team lead / Thread 01 — copy into new Codex task
You are Thread 01, the architecture and integration lead for StreamPulse (https://github.com/Thilac01/StreamPulse). Read README.md, AGENTS.md, TEAM_CODEX_THREADS.md and technical_architecture.json. If a file is missing, say so. PLAN ONLY: do not build, create application source code, install anything, connect external services, or deploy. Audit repo readiness; define the MVP boundary, complete cross-service message contracts, six-thread ownership, integration order, review checkpoints, blockers, and a 3-minute demo acceptance checklist. Produce a concise planning response. Wait for explicit START BUILD.

## Thread 02 — Twitch integration
You own Twitch authentication, EventSub, chat vote ingestion, reconnection and deduplication. Read the four planning files. PLAN ONLY. Propose OAuth scopes, Twitch developer setup, event delivery mode, secure token management, chat parsing rules, test channel setup, reliability cases, and shared input/output contracts. Identify API details needing verification. Do not implement. Wait for START BUILD.

## Thread 03 — Racing moments and timing
You own racing simulator/event adapter and the ask-vs-stay-silent timing gate. Read the planning files. PLAN ONLY. Propose event schema, duplicate suppression, thresholds, cooldowns, prompt templates, confidence rules, and deterministic replay fixtures. Define acceptance tests and service boundaries. No implementation. Wait for START BUILD.

## Thread 04 — Dashboard and overlay
You own the streamer controls, chat-vote results display, browser/OBS overlay and frontend real-time client. Read planning files. PLAN ONLY. Define UI screens/states, accessibility, responsive layout, WebSocket/SSE event schema, loading and recovery states, and race overlay safe zones. No implementation. Wait for START BUILD.

## Thread 05 — Polls, outcomes and scoring
You own vote lifecycle, one-vote-per-user enforcement, event outcome reconciliation and session points/leaderboard. Read planning files. PLAN ONLY. Define API/database schema and concurrency/idempotency rules, demo fixtures, error cases and test matrix. No implementation. Wait for START BUILD.

## Thread 06 — Quality, performance and demo
You own integration test strategy, latency measurement, failure recovery and three-minute demo. Read planning files. PLAN ONLY. Define reproducible Twitch/test-channel checks, simulator test plan, metrics instrumentation, security checklist, smoke tests, fallback demo and launch checklist. No implementation. Wait for START BUILD.

## Before starting work
All members must confirm who owns shared interfaces (lead 01) and agree on contracts before implementation. Use independent Codex Cloud tasks or separate local Git worktrees/feature branches. Do not copy one task's uncommitted changes into another task automatically.
