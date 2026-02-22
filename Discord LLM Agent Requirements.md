# Discord LLM Agent Requirements (Local-First)

Updated: 2026-02-21  
Scope root: `05 - Projects/Billi`

## 1. Project Summary
- Build a local-first LLM agent for Discord.
- The agent must be reachable in Discord via mention in text channels.
- The agent must be controllable from VS Code through MCP tools.
- The runtime must be 24/7 on local infrastructure (your machine), using free-tier AI providers in phase 1.

## 2. Goals
- Provide ChatGPT-like interaction for Discord operations.
- Continuously monitor specific Discord channels (log/watch channels).
- Execute server actions only through explicit policy-controlled tools.
- Keep mention commands restricted and safe.
- Maintain full audit logs of actions and decisions.

## 3. Non-Goals (Phase 1)
- No self-bot or user-account automation.
- No paid cloud dependencies.
- No unrestricted autonomous actions.
- No full voice-call conversation pipeline in phase 1 (voice can be phase 2).

## 4. Core Constraints
- Primary LLM provider: Gemini API (free tier).
- Secondary LLM provider: OpenRouter (routing/fallback).
- Chat routing may use high-throughput OpenRouter pools.
- Tool execution must use pinned stable models only.
- Local model fallback (Ollama) is deferred to a later phase.
- Discord operations must use official bot account permissions only.
- Implementation coding is done by owner; architecture/planning support only until requested.

## 5. System Architecture Requirements
- `Discord Bot Runtime`: always-on process connected to Discord Gateway.
- `LLM Layer`: provider router with separate execution and chat lanes.
- `Policy Guard`: validates who asked, where, and whether action is allowed.
- `Command Router`: routes mention commands and MCP commands.
- `Action Executor`: performs approved Discord operations.
- `State/Audit Store`: SQLite for configs, logs, and confirmations.
- `MCP Server`: exposes controlled tools for VS Code chat/agent use.
- `Operator Interface`: VS Code chat as control surface, not the always-on runtime.

## 6. Functional Requirements
- `FR-001` The bot must run continuously and auto-recover after crashes.
- `FR-002` The bot must process mentions in allowlisted channels.
- `FR-003` The bot must listen to configured log/watch channels continuously.
- `FR-004` Mention-based actions must be restricted to an allowlisted command set.
- `FR-005` MCP-based actions must be owner-only by default.
- `FR-006` Every action must pass guild/channel/role/user policy checks.
- `FR-007` The agent must support safe read operations (channel/user/status inspection).
- `FR-008` The agent must support allowed write operations (send/edit/delete messages) based on policy.
- `FR-009` High-risk actions (kick/ban/role changes/channel deletes) must require explicit confirmation.
- `FR-010` The system must provide clear denial messages when policy blocks a command.
- `FR-011` All tool calls and resulting Discord actions must be logged with timestamp, actor, source, and outcome.
- `FR-012` The system must store and use per-guild configuration for watch channels and permissions.
- `FR-013` The system must support a dry-run/preview mode for high-risk actions.
- `FR-014` The system must expose health/status for runtime verification.
- `FR-015` The system must support rate-limit-aware retries and failure handling.

## 7. Non-Functional Requirements
- `NFR-001` Cost: 100% free runtime stack (local-only).
- `NFR-002` Security: secrets stored outside source files (environment/secure config).
- `NFR-003` Reliability: startup on boot + automatic restart policy.
- `NFR-004` Observability: structured logs and actionable error reporting.
- `NFR-005` Performance: acceptable mention response latency on local hardware.
- `NFR-006` Compliance: no Discord policy violations (no self-bot behavior).
- `NFR-007` Maintainability: modular separation (policy, routing, execution, model).

## 8. Permission and Safety Model
- Actor: `Owner` with full MCP control within policy.
- Actor: `Discord users` with restricted mention command surface.
- Source: `mention` is strict allowlist and lower privilege.
- Source: `mcp` supports broader admin operations but remains policy-constrained.
- Risk tier: `Low` for read/list/info operations.
- Risk tier: `Medium` for message write/edit/delete operations.
- Risk tier: `High` for moderation or structural changes.
- Safety gate: explicit confirmation token for high-risk actions.
- Safety gate: optional dry-run output before execution.
- Safety gate: full audit trail for all mutating actions.

## 9. Data Requirements (Conceptual)
- Guild configuration (guild IDs, allowed channels, enabled features)
- Watch channel registry
- Command policy definitions (source + actor + action)
- Audit log of executed and blocked actions
- Pending confirmations for high-risk operations
- Model/runtime settings (provider, lane, model list, timeout, retry, safety flags)

## 10. Operations Requirements
- Start automatically on machine boot.
- Restart on process failure.
- Keep runtime and audit logs locally.
- Provide a quick health check command/workflow.
- Provide backup/restore strategy for SQLite state.

## 11. Roadmap
- `Phase 0` Foundation: runtime skeleton, policy guard, local persistence, health checks.
- `Phase 1` Text + Control: mentions in text channels, watch channels, MCP control path, audit logging.
- `Phase 2` Reliability: add local LLM fallback (Ollama) and provider resilience hardening.
- `Phase 3` Voice: voice channel join/listen/speak pipeline (STT/TTS), gated by strict policy.
- `Phase 4` Hardening: richer guardrails, quality improvements, prompt/tool optimization.

## 12. Acceptance Criteria
- Bot remains active and reconnects automatically when disconnected.
- Mention command in allowed channel returns response and executes only approved actions.
- Mention command in disallowed channel is denied with clear reason.
- MCP command from VS Code executes allowed action and writes audit entry.
- High-risk command cannot execute without confirmation flow.
- Rebooting local machine restores the bot automatically.

## 13. Open Decisions to Finalize
- Exact list of allowed mention commands.
- Exact list of MCP tools and which are high-risk.
- Guild/channel/user allowlists for first deployment.
- Whether moderation actions are enabled in phase 1 or phase 2.
- Retention period for logs and conversation context.
- Exact pinned execution models (Gemini primary, OpenRouter fallback A/B).
- Exact chat pool model list and throughput routing policy.
- Voice feature priority and scope.

## 14. Immediate Next Step
- Produce a strict `Command and Permission Matrix` document.
- Required columns: `command`, `source`, `allowed actor`, `allowed channels`, `risk tier`, `confirmation required`, `audit fields`.

## 15. Linked Policy
- `docs/LLM-Provider-Policy.md`
- `docs/Action-Catalog.md`
- `docs/Command-Permission-Matrix.md`
- `docs/Data-Context-Strategy.md`
