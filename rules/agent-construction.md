---
alwaysApply: true
---

# Agent Construction

## Use the top-level `AIAgent(...)` factory

- In 1.0, `AIAgent` is an `expect abstract class` with three concrete subtypes — `GraphAIAgent`, `FunctionalAIAgent`, `PlannerAIAgent`. Don't construct these directly
- The supported entry point is the top-level factory function `AIAgent(...)` in `ai.koog.agents.core.agent.AIAgentFactory` — it returns the right concrete subtype for the strategy you pass in
- The old companion-`invoke` constructor (`AIAgent.invoke(...)`) was removed in 1.0 (#1882). The call site `AIAgent(...)` looks unchanged but it now resolves to the top-level factory

## Pass features through the trailing lambda

- The factory's last parameter is `installFeatures: GraphAIAgent.FeatureContext.() -> Unit = {}` — install OpenTelemetry, event handlers, long-term memory, persistence, and other features inside that block
- Do not install features by mutating the agent after construction — the features API expects the install hooks to run during construction so the pipeline wires up correctly

## `singleRunStrategy()` is the default

- The single-step "call LLM, run tools if asked, loop until text reply" strategy is `singleRunStrategy(parallelTools: Boolean = false)`. It's the default; omit it from the call unless you're overriding it
- Authoring a custom strategy is a separate workflow — use the `author-strategy` skill

## Don't hardcode `maxIterations`

- `maxIterations` defaults to 50 in the factory. Override it only when the strategy or planner has a clear bound — uncapped loops are how agentic code burns money in production
- Planner agents need much higher caps (the in-repo example uses 400) — each step is at least one LLM round-trip

## Identify and clock

- Pass `id` for production agents so traces and persistence records correlate across runs — auto-generated IDs make logs unreadable
- Use `KoogClock.System` (the default) in production; inject a fake `KoogClock` in tests so deterministic test runs don't drift on real wall time

## `AgentMemory` was removed in 1.0

- Don't install an `AgentMemory` feature — it doesn't exist anymore. Use `LongTermMemory` for cross-session memory; reach for it via the `manage-state` skill
- Most agents don't need either — agent-run state lives on `AIAgentContext.storage` automatically and is serialized into checkpoints

## When to reach for a skill

- Authoring a custom graph strategy → `author-strategy`
- Using a planner (LLM-based or GOAP) → `use-planner`
- Working with `storage`, history compression, or `LongTermMemory` → `manage-state`
- Adding a tool → `add-tool`; connecting to an MCP server → `wire-mcp-server`
- Installing OpenTelemetry → `add-observability`
- Spring Boot autoconfig → `wire-spring-boot`; Ktor plugin → `wire-ktor-server`
- Migrating 0.x code to 1.0 → `migrate-from-0-x`
