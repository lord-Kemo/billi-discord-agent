# LLM Provider Policy (Phase 1)

Updated: 2026-02-21  
Project: `billi-discord-agent`

## 1. Purpose
- Define how the agent chooses LLM providers safely and reliably.
- Separate conversational quality routing from tool-execution safety.
- Keep the system operational under free-tier limits and temporary outages.

## 2. Provider Strategy
- Primary provider: `Gemini API` (direct integration).
- Secondary provider: `OpenRouter` (fallback and routing layer).
- Local LLM: deferred to later phase (not part of phase 1 runtime path).

## 3. Two-Lane Model Policy
- `Execution lane` (tool calls and server actions) uses a small pinned model set only.
- `Execution lane` prioritizes stability and tool-call consistency over speed.
- `Execution lane` does not use high-throughput pool rotation.
- `Chat lane` (general assistant conversation) uses OpenRouter throughput routing with a larger model pool.
- `Chat lane` prioritizes responsiveness and availability.

## 4. Execution Lane Rules
- Allowed providers:
- Tier 1: Gemini pinned execution model.
- Tier 2: OpenRouter pinned fallback model A.
- Tier 3: OpenRouter pinned fallback model B.
- If execution model confidence/format checks fail, block action.
- Require confirmation token before all high-risk mutating actions.
- Never execute high-risk actions from a non-pinned throughput model.

## 5. Chat Lane Rules
- OpenRouter can route through a high-throughput pool (up to 10 models).
- Throughput routing is allowed only for non-critical chat outputs.
- Chat lane cannot directly execute tools without passing through execution lane policy checks.

## 6. Failover Policy
- Failure trigger: rate limit (`429`).
- Failure trigger: timeout.
- Failure trigger: provider unavailable (`5xx`).
- Failure trigger: invalid structured output for required tool format.
- Retry once on the same provider for transient timeout.
- Then fail over: Gemini -> OpenRouter pinned fallback A -> fallback B.
- If all execution models fail, return safe failure message and do not execute.
- Chat lane may degrade to smaller/cheaper models before returning failure.

## 7. Safety and Guardrails
- Tool calls are allowed only from execution lane.
- Tool call payload must pass schema validation before action.
- Any policy violation returns explicit denial and audit log entry.
- Mutating operations require actor, target, reason, and source recorded.

## 8. Rate-Limit and Quota Handling
- Track per-provider request counters and recent error rate.
- Apply per-lane throttles.
- Execution lane gets reserved capacity budget.
- Chat lane is throttled first during quota pressure.
- If quota is near limit, switch chat lane to lighter models automatically.

## 9. Observability Requirements
- Log fields for every request: `request_id`, `lane`, `provider`, `model`, `source`, `latency_ms`, `status`, `error_code`.
- Log fields for every attempted action: `actor`, `action`, `target`, `risk_tier`, `confirmation_state`, `result`.
- Dashboard minimum: success rate by provider.
- Dashboard minimum: failover count.
- Dashboard minimum: blocked action count.
- Dashboard minimum: quota/rate-limit events.

## 10. Configuration Contract (Conceptual)
- `LLM_EXECUTION_PRIMARY` (Gemini model ID)
- `LLM_EXECUTION_FALLBACK_A` (OpenRouter model ID)
- `LLM_EXECUTION_FALLBACK_B` (OpenRouter model ID)
- `LLM_CHAT_POOL` (OpenRouter model IDs list)
- `LLM_EXEC_TIMEOUT_MS`
- `LLM_CHAT_TIMEOUT_MS`
- `LLM_MAX_RETRIES`
- `LLM_DISABLE_CHAT_POOL` (emergency flag)

## 11. Phase Scope
- Phase 1: Gemini + OpenRouter only.
- Phase 1: no local model fallback.
- Phase 2+: add local Ollama fallback for outage resilience.
- Phase 2+: optionally move low-risk chat to local model to reduce external dependency.

## 12. Acceptance Criteria
- Execution requests always use pinned execution models.
- High-throughput chat models never directly execute tools.
- Provider failure causes controlled failover without unsafe actions.
- High-risk actions are blocked if confirmation is missing or provider output is invalid.
- All provider calls and action attempts are auditable.
