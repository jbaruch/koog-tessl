---
alwaysApply: true
---

# Module Coordinates

## Use 1.0+, not 0.x

- All Koog artifacts ship under group `ai.koog`. Pin to `1.0.0` or later
- Never mix 0.x and 1.0 artifact versions in one project — the API surface diverged at 1.0 (factory functions, planner module split, HTTP transport decoupling); a mixed graph compiles in unpredictable ways and fails at link time
- The hosted Maven snippet on `docs.koog.ai/quickstart/` still showed `0.7.1` at the time this rule was written. Don't copy it — use `1.0.0` or later

## Start with the umbrella

- `ai.koog:koog-agents:1.0.0` is the umbrella that pulls everything needed for the README example: agent core, the simple executor factories, common providers
- Reach past the umbrella only when you need something it does not pull: the planner, MCP, Spring Boot, Ktor, OpenTelemetry, long-term memory

## Pull these explicitly when used

- `ai.koog:agents-planner` — the planner DSL (`Planners.llmBased`, `Planners.goap`) lives in its own module since 1.0
- `ai.koog:agents-mcp` (client) / `ai.koog:agents-mcp-server` (server) — MCP support
- `ai.koog:koog-spring-boot-starter` — autoconfig for Spring Boot
- `ai.koog:koog-ktor` — Ktor plugin
- `ai.koog:agents-features-opentelemetry` — observability
- `ai.koog:agents-features-longterm-memory` — cross-session memory (and `*-longterm-memory-aws` for Bedrock AgentCore backend)
- `ai.koog:prompt-executor-<provider>-client` — only when you want a provider that the umbrella does not bundle (e.g., DashScope, LiteRT)

## HTTP transport is now a choice

- The 1.0 HTTP transport is decoupled from Ktor (#2006). On the JVM the default `KoogHttpClient.Factory` is auto-discovered, so `simpleOpenAIExecutor(apiKey)` still works without naming a transport
- If you want a non-default transport, pull one of `ai.koog:http-client-ktor`, `ai.koog:http-client-okhttp`, `ai.koog:http-client-java` and pass its factory to the executor
- Do not depend transitively on Ktor types via `prompt-executor-llms-all` — that route was closed in 1.0; declare `http-client-ktor` directly if your code references Ktor

## JDK and tooling minima

- JDK 17 minimum (#1931). Targeting JDK 8 or 11 will not compile against 1.0
- Android consumers must set `android.useAndroidX=true` in `gradle.properties`
