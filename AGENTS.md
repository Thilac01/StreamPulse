# StreamPulse Codex Instructions

These instructions apply to the entire repository.

## Current mode: PLAN ONLY

Do not create application source code, install packages, initialize frameworks, modify infrastructure, deploy, or connect to live production services. Planning-document edits are allowed.

Implementation begins only when the team lead writes the exact phrase:

`START BUILD`

When authorization is absent or ambiguous, stop after planning and report the missing decision.

## Product guardrails

- Build the racing-first Twitch MVP described in `README.md`.
- Optimize for a dependable three-minute demonstration.
- Treat the deterministic simulator as an MVP dependency; telemetry and AI are optional adapters.
- Disclose simulated inputs. Never present fixtures or mocked Twitch responses as live data.
- Predictions use session entertainment points only—no money or monetary prizes.
- Do not make a reviewed Twitch Extension a prerequisite. The MVP is a web companion, chat integration, and browser-source overlay.
- Streamers must be able to pause, resume, and skip interactions.

## Coordination rules

- Read `README.md`, `TEAM_CODEX_THREADS.md`, `technical_architecture.json`, and both files in `docs/` before proposing changes.
- Work only inside the file ownership assigned to the task unless the integration owner approves a handoff.
- Shared contracts are owned by Member 1. Other members propose contract changes rather than editing them unilaterally.
- Use one branch/worktree per member: `m1/integration`, `m2/twitch`, `m3/events`, `m4/frontend`, `m5/voting`, `m6/quality`.
- Keep commits small and name the affected role or contract.
- Never overwrite another member's changes. Surface conflicts to Member 1.
- Integrate vertically in thin slices: simulator event → timing decision → question → chat vote → overlay → outcome → score.

## Security and privacy

- Never commit Twitch client secrets, OAuth tokens, broadcaster credentials, cookies, personal access tokens, or real chat exports.
- Use `.env` locally and commit only a redacted `.env.example` after build authorization.
- Use OAuth state and PKCE where applicable, validate redirect URIs, and keep tokens server-side.
- Request the least Twitch scopes needed and verify them against current official Twitch documentation before implementation.
- Validate EventSub signatures or session identity as required by the selected transport.
- Minimize retention of viewer identifiers; use scoped/pseudonymous identifiers for session scoring.
- Redact secrets and viewer content from logs and screenshots.
- Rate-limit control endpoints and make state transitions idempotent.

## Contract rules

- Use UTC ISO 8601 timestamps and opaque string IDs.
- Every cross-service event carries `schemaVersion`, `eventId`, `occurredAt`, and `correlationId`.
- Do not rename frozen event types without an architecture decision and coordinated migration.
- Consumers must tolerate duplicate delivery and reject unsupported schema versions clearly.
- The server is authoritative for question state, vote eligibility, resolution, and scores.

## Required task report

For each assignment, report:

1. Goal
2. Files affected
3. Proposed approach
4. Dependencies and contract changes
5. Tests and validation
6. Risks and mitigations
7. Status: `PLANNED`, `BLOCKED`, or—only after authorization—`BUILT`

## Definition of done after build authorization

- Owned unit and integration tests pass.
- Contract fixtures validate against shared schemas.
- Failure and retry behavior is demonstrated.
- No secrets or personal data are present in code, logs, fixtures, or screenshots.
- Member 1 verifies the vertical demo path and Member 6 verifies the fallback path.
- Documentation reflects the implemented behavior.

