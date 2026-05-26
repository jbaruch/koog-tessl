# Changelog

All notable changes to this tile are documented here. Format: [Keep a Changelog](https://keepachangelog.com/en/1.1.0/). Versioning: [SemVer](https://semver.org/).

## [0.4.4] — 2026-05-26

### Changed

- Hardened skills against the **file-write** failure mode observed in 0.3.1 eval run `019e60f5` and confirmed in the partial re-run `019e613e`: the scorer reads files from the solution directory, but several skills told the agent "produce ... as part of your response" — which the agent satisfied with stdout prose that the scorer can't see. Adopted the `Path: <file>` convention from `scaffold-agent` across the patched skills so the file targets are explicit and unambiguous:
  - `add-observability` Step 3 — `Path: Main.kt` for the modified agent construction, `Path: build.gradle.kts` for the dependency
  - `manage-state` Step 2 — `Path: Strategy.kt` for the boundary-node body
  - `persist-chat-history` Step 4 — `Path: Main.kt` + `Path: build.gradle.kts` + a concrete `Path: <route-file>` when the user named a handler
  - `add-tool` Step 2 — `Path: <ToolName>.kt` (concrete tool-name example, not a literal placeholder) + `Path: Main.kt` + `Path: build.gradle.kts`
  - `author-strategy` Step 8 — `Path: Strategy.kt` for the DSL + `Path: Main.kt` for the modified construction
  - `handle-agent-events` Step 2 — same `Path:` convention applied
  - `wire-ktor-server` Step 5 — `Path: Application.kt` for the modified module + `Path: build.gradle.kts`

- `add-tool` Step 1 routing — annotated-tool default now defers to Step 2 (typed `Tool<TArgs,TResult>`) when the user's existing function takes a `data class` parameter or returns a typed result; the previous "default to Step 1" rule wrapped typed signatures in flat-primitive annotated tools and lost the type contract

- `handle-agent-events` Step 2 — adopted the same `Path:` directive; round-2 eval `019e6149` showed the prior prose-only handoff was non-deterministic (round 1: 100, round 2: 0)

- `use-planner` Step 1 redirect to `author-strategy` — made the redirect actionable: it now runs `author-strategy` end-to-end and writes the graph DSL code via `Path:` per author-strategy's Step 8, plus topology and round-trip-cost reasoning as comments at the top of the produced file; the previous wording let the agent stop at a prose explanation. Step 1 also adds a "Finish here — do not continue into planner-variant selection or Step 2 / Step 3" line so the redirect and planner-fits branches are mutually exclusive, and an explicit "Chaining exception (exhaustive — overrides 'Do not run other steps' only as listed)" preamble names the Step 1 → Step 2/3 chain per `skill-authoring`

- Hardened skills against the "no-output" failure mode observed in 0.3.1 eval run `019e60f5`:
  - `add-observability` Step 2 — replaced blocking "Ask the user which backend" with a non-blocking pick + OTLP default; Step 3 writes via `Path:`; fixes the −100pp lift on `add-observability-langfuse`
  - `wire-ktor-server` Step 2 — split into minimal install (mandatory) vs MCP/HOCON add-ons; Steps 3 and 4 each open with "Skip this step entirely if..."; Step 5 writes via `Path:`; fixes the −28pp lift on `wire-ktor-server-route`
  - `manage-state` Step 2 — committed to `HistoryCompressionStrategy.WholeHistory` as the default and moved the other six variants into "use only when the user names them"; fixes the −80pp lift on `manage-state-tldr-mid-phase`
  - `use-planner` Step 1 — converted the "ask user LLM-based vs GOAP" stall into a pick-by-keywords rule (GOAP only when the user names typed state / classical planner / state space); planner-redirect tasks no longer block on a clarifying question
- Tightened skill activation routing for planner construction:
  - `use-planner` description now names `Planners.llmBased`, `Planners.llmBasedWithCritic`, `Planners.goap`, `PlannerAIAgent`, `agents-planner`
  - `scaffold-agent` description adds an explicit "Do NOT use when the user is constructing a planner / picking a strategy / naming a specific agent shape" exclusion; fixes the mis-activation that pulled `scaffold-agent` for `use-planner-llm-based-triage`

### Removed

- `cache-llm-calls-redis-shared` eval scenario — retired per `plugin-evals.md` "Lift, Not Attainment": baseline 100/100, lift 0pp (Cause #1, universal competence). The partner scenario `cache-llm-calls-refuses-provider-side` still covers `cache-llm-calls` at +100pp lift

### Fixed

- `handle-agent-events-stdout-trace` task — stripped the "arrow indicating start vs end" framing that bled into the criterion "Uses distinct visual markers for start vs end" (`plugin-evals.md` "No Bleeding")
- `add-tool-typed-args-with-result` task — stripped "they want the tool's input and output to remain these typed shapes — not a JSON blob, not a flattened String" framing that telegraphed `Tool<TArgs,TResult>`; the typed `queryAccount` signature still carries the constraint
- `persist-chat-history-jdbc` task — replaced "wants conversations to persist by user account" framing with a user-reported bug ("the bot doesn't remember anything we talked about") so the agent must navigate persistence-feature vs chat-history vs LongTermMemory on its own

## [0.4.3] — 2026-05-26

### Fixed

- Stripped XML-tag-looking syntax from skill `description:` fields in `add-tool` (`Tool<TArgs,TResult>` → `Tool[TArgs,TResult]`; `<X>` / `<function>` → prose), `domain-model-subtask-pipeline` (`subgraphWithTask<In, Out>` → `subgraphWithTask[In, Out]`; same for `subgraphWithVerification<T>` and `CriticResult<T>`), `use-llm-node-variants` (`<tool>` → prose). The `tessl skill review` validator rejects `<` followed by alpha as an XML tag — the just-landed skill-review CI gate (0.4.1) failed `add-tool` on the previous 0.4.2 publish attempt because of this. **The 0.4.2 publish never landed in the registry** (failed at the gate); 0.4.3 ships the same content as 0.4.2 plus this descriptor fix

## [0.4.2] — 2026-05-26 (never published)

### Fixed

- `author-strategy` skill — added a member-vs-extension import table at the end of Step 5. The DSL primitives split across two shapes: `forwardTo` / `onCondition` / `transformed` are infix members (no import needed); `onToolCalls` / `onTextMessage` / `onIsInstance` / `onSuccessful` / `onFailure` / `asUserMessage` / `asToolResultMessage` / `onMessageParts` are top-level extensions in `ai.koog.agents.core.dsl.extension.*` (each needs its own import). Inventing a member import or omitting an extension import is the most common copy-paste compile failure
- `author-strategy` + `domain-model-subtask-pipeline` skills — fixed the wrong artifact text. `subgraphWithTask` / `subgraphWithVerification` / `CriticResult` (package `ai.koog.agents.ext.agent`) ship inside `agents-core` (which `koog-agents` umbrella pulls), NOT the standalone `ai.koog:agents-ext:1.0.0-beta` artifact. Added the imports + the `AIAgentGraphStrategy` package note (`ai.koog.agents.core.agent.entity`, not bare `ai.koog.agents.core.agent`)
- `wire-mcp-server` skill — each transport builder (`streamableHttp`, `fromSseUrl`, `fromProcess`) is a top-level extension in `ai.koog.agents.mcp` declared separately from the `McpToolRegistryProvider` object. All three example blocks now show the explicit extension import alongside the provider import
- `wire-mcp-server` skill + `module-coordinates` rule — corrected the MCP client dependency to `ai.koog:agents-mcp-jvm:1.0.0-beta`. Koog 1.0 stable did not publish `agents-mcp` / `agents-mcp-server` at `1.0.0`; only `1.0.0-beta` is on Maven Central, and they publish only JVM variants so the `-jvm` suffix is mandatory for Gradle KMP variant resolution
- `add-tool` + `wire-a2a` skills — corrected the `ToolSet` and `asTools` imports to come from `ai.koog.agents.core.tools.reflect.*` (the actual package), not bare `ai.koog.agents.core.tools.*`
- `module-coordinates` rule — added Kotlin 2.3.10+ minimum requirement. Koog 1.0 is compiled with Kotlin 2.3.x; earlier Kotlin versions fail at consume time with metadata-version errors
- `evals/domain-model-subtask-pipeline-triage/criteria.json` — corrected C7's required-Gradle-deps description: the umbrella `ai.koog:koog-agents:1.0.0` is sufficient (it pulls `agents-core`, which contains the subgraph DSL). Penalize unnecessary `agents-ext` lines

### Added

- `evals/author-strategy-import-shapes` — new positive scenario testing the member-vs-extension import correctness when the agent emits a tool-handling-loop strategy
- `evals/wire-mcp-server-import-shapes` — new positive scenario testing the `fromSseUrl` extension import + the `-jvm:1.0.0-beta` dependency line. Updated `wire-mcp-server-stdio-playwright`, `wire-mcp-server-streamable-http`, and `wire-mcp-server-merge-tools` criteria to match the corrected artifact spec

Closes #9 (items 1–9; item 10 is a separate Tessl install-policy investigation).

## [0.4.1] — 2026-05-26

### Added

- `.github/workflows/publish.yml` — `tessl skill review --threshold 85` gate before `tessl tile publish .` via `jbaruch/coding-policy/.github/actions/skill-review@ef67ffe5` (changed-skills loop). Closes the `context-artifacts` Mandatory Review gap flagged on #7; below-threshold skill scores now block publish. Checkout step bumped to `fetch-depth: 0` so the action's `git diff $github.event.before..HEAD` can resolve the prior commit. Closes #8

## [0.4.0] — 2026-05-25

### Added

- `domain-model-subtask-pipeline` skill — the integrated pattern for typed-handoff pipelines: tools sliced by access into separate `ToolSet`s (read / write / communication), `@Serializable` `@LLMDescription`-annotated data classes as inter-subtask contracts, `subgraphWithTask<In, Out>` per phase with per-phase model selection, `subgraphWithVerification<T>` + `CriticResult<T>` for self-correction loops. The methodology JetBrains' KotlinConf 2026 banking demo demonstrates — fills the gap left by `author-strategy` (DSL mechanics only) and `add-structured-output` (top-level typed output only)
- 2 eval scenarios — `domain-model-subtask-pipeline-triage` (positive, four-phase support workflow) and `domain-model-subtask-pipeline-refuse` (negative — declines to over-engineer a one-shot text transform)

## [0.3.1] — 2026-05-25

### Added

- `.github/workflows/publish.yml` — `tesslio/patch-version-publish@v1` wired to push-on-main + manual dispatch; auto-bumps patch from the registry on future merges
- `.github/workflows/review-openai.md` / `review-anthropic.md` — paired gh-aw PR reviewers from `jbaruch/coding-policy: install-reviewer` (cross-family review per `author-model-declaration`)
- `.env.example` — required hosted-CI secrets with placeholder values and a deep link to `https://github.com/jbaruch/koog-tessl/settings/secrets/actions` (per `no-secrets`)
- `.pre-commit-config.yaml` — gitleaks v8.21.2 + standard pre-commit-hooks (per `no-secrets` pre-commit-scanning requirement)
- 4 negative eval scenarios — `use-planner-refuses-when-graph-fits`, `cache-llm-calls-refuses-provider-side`, `add-persistence-refuses-conversation`, `snapshot-and-restore-refuses-crash` — covering the cross-skill redirects each skill prescribes (closes the "only positive cases" gap from 0.3.0)
- 1 generator-produced eval scenario — `use-attachments-pdf-and-url` (PDF + URL-image attachments in a single LLM call); the existing `use-attachments-image-input` only exercised images
- `scenario.json` backfilled on every eval scenario (40 total) to match the canonical three-file shape `tessl scenario generate` emits — drift fix, not a feature

### Changed

- Slimmed `tile.json` `summary` from a 290-char comma-spliced multi-clause string back to a one-line description per `skill-authoring.md`
- Stripped `interaction-rules` phantom reference from 8 skill files (the rule never existed in this tile or in any consumed tile)
- Converted 7 prose skill cross-references (`see the \`X\` skill`, `redirect to \`X\``) to typed `Skill(skill: "X")` calls per `skill-authoring.md` "Typed Calls"
- Split 6 step titles that combined verbs with "and" (`Verify and Hand Off`, `Bump JDK and Tooling`, etc.) per `skill-authoring.md` "Step Structure"
- Reworded `model-planner-subtasks-parallel-tree` task to remove technique leak (`PlannerNode composition`, `storage-key tracking`) per `plugin-evals.md` "No Bleeding"
- Reworded `use-attachments-pdf-and-url` task on the cross-family reviewer's finding — dropped `strategy` / `user message` technique prose
- Renamed `enable-prompt-caching-anthropic-long-system` (43 chars) → `enable-prompt-caching-anthropic` (31) to fit the 40-char default cap
- Renamed `wire-mcp-{merge-with-existing-tools,stdio-playwright,streamable-http-github}` → `wire-mcp-server-{merge-tools,stdio-playwright,streamable-http}` so prefixes match the skill name per `plugin-evals.md` "Naming"

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
