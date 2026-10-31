# StreamPulse — Codex repository instructions

## Status and authorization
PLANNING ONLY. Do not generate application source files, install dependencies, run migrations, provision infrastructure, connect credentials, or deploy until the team lead explicitly writes `START BUILD` in the relevant task. This rule applies to all six agents.

## Context (read before acting)
1. `README.md` — product and MVP scope.
2. `TEAM_CODEX_THREADS.md` — ownership and collaboration.
3. `technical_architecture.json` — proposed stack, interfaces, data, latency, security.
If a referenced file is absent, report it; do not fabricate its contents.

## Product boundaries
Racing-first Twitch audience engagement. End-to-end flow: racing event -> timing gate -> prediction -> viewer chat vote -> results overlay -> verified outcome -> points. A simulator is permitted for racing input but must be clearly labeled; never claim simulated events are automatically detected from video. Viewer chat must use actual Twitch integration in the real demo.

## Team coordination
Six independent Codex tasks/workspaces. Members own: 01 architecture/integration, 02 Twitch gateway, 03 racing event/timing, 04 frontend/overlay, 05 polls/scoring, 06 tests/performance/demo. Keep changes scoped to assigned paths once implementation is authorized. Define shared contracts before coding. Submit changes through branches and pull requests; do not directly change `main`. Avoid editing other members' code without requesting coordination.

## Planning task output
In planning mode, inspect repo contents; describe a precise, ordered plan, dependencies and API contracts, assumptions, risks, acceptance tests, and files your member would own. Do not create code or execute installation commands. Ask the team lead about material blockers only.

## Build gates
Only after explicit `START BUILD`, implement approved scope in small PRs with tests and documentation; never commit credentials, tokens, `.env` files or user chat data. Production OAuth scopes and EventSub subscription types must be verified against Twitch docs before integration. Use actual measurements rather than invented latency claims.
