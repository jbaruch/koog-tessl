# Changelog

All notable changes to this tile are documented here. Format: [Keep a Changelog](https://keepachangelog.com/en/1.1.0/). Versioning: [SemVer](https://semver.org/).

## [0.3.0] — 2026-05-25

### Added

- 7 additional skills covering modules and API surfaces missed in 0.2.0:
  - `cache-llm-calls` — in-process LLM-response cache (`prompt-executor-cached` + `prompt-cache-{files,model,redis}`), distinct from the provider-side caching covered by `enable-prompt-caching`
  - `persist-chat-history` — chat-history persistence backends (`chat-history-jdbc`, `chat-history-aws`, `chat-memory-sql`), distinct from generic persistence and `LongTermMemory`
  - `test-koog-agents` — deterministic agent testing with `agents-test` (scripted executor, fake `KoogClock`, event-handler recorder)
  - `trace-agent-internals` — deep diagnostic trace feature (`agents-features-trace`), distinct from OpenTelemetry (production signal) and event handlers (high-level callbacks)
  - `query-sql-from-agent` — SQL-querying feature (`agents-features-sql`) with read-only mode, schema scoping, row caps
  - `model-planner-subtasks` — `PlannerNode` tree composition, parallel vs sequential subtasks, retry-on-parse-failure edges, history compression between phases
  - `use-functional-agent` — `FunctionalAIAgent` (the third concrete agent subtype, alongside `GraphAIAgent` and `PlannerAIAgent`) — single suspending block, no graph
- 7 new eval scenarios — 1 per new skill, weighted-checklist with non-uniform weights summing to 100
- Scope statement in README clarified: Kotlin/JVM only; Kotlin/JS, Kotlin/Native, Compose Multiplatform explicitly out of scope

## [0.2.0] — 2026-05-25

### Changed (breaking)

- **Slimmed rules from 9 to 2.** Only `module-coordinates` and `agent-construction` remain always-on — they cover gotchas every Koog project hits. The other 7 rules were converted to on-demand skills:
  - `strategy-dsl` → `author-strategy` skill
  - `planner-vs-graph` → `use-planner` skill
  - `tools-and-mcp` → folded into `add-tool` and `wire-mcp-server` skills (already existed)
  - `state-and-memory` → `manage-state` skill
  - `observability` → `add-observability` skill
  - `spring-boot-integration` → `wire-spring-boot` skill
  - `migration-from-0-x` → `migrate-from-0-x` skill
- Front-loaded token cost dropped from ~6.9k to ~1.3k

### Added

- 19 new skills filling Koog 1.0 surface gaps not covered in 0.1.0:
  - High priority: `add-structured-output`, `define-prompt`, `add-persistence`
  - Medium priority: `enable-prompt-caching`, `handle-agent-events`, `wire-ktor-server`
  - Lower priority but real coverage: `use-llm-node-variants` (streaming / multiple-choices / moderation / force-one-tool), `add-rag`, `wire-a2a`, `wire-acp-server`, `add-token-budgeting`, `snapshot-and-restore`, `use-attachments`
- 19 new eval scenarios — 1 per new skill, all weighted-checklist with non-uniform weights summing to 100
- `agent-construction` rule now includes a "When to reach for a skill" index pointing to the right skill for each common task

## [0.1.0] — 2026-05-25

### Added

- Initial tile targeting Koog 1.0.0 (released 2026-05-21)
- 9 always-apply rules covering module coordinates, agent construction, the strategy DSL, planner vs graph, tools & MCP, state & memory, observability, Spring Boot integration, and the 0.x → 1.0 migration surface
- 3 skills: `scaffold-agent`, `add-tool`, `wire-mcp-server`
- 9 eval scenarios (3 per skill)
- Kotlin-only scope; Java-interop surface deferred to a future sibling tile
