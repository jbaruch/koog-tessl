---
name: persist-chat-history
description: >
  Persist a Koog 1.0 agent's chat history to a durable backend — JDBC database
  (`chat-history-jdbc`), AWS storage (`chat-history-aws`), or SQL-typed chat memory
  (`chat-memory-sql`) — so conversations survive process restarts and can be retrieved
  by session ID. Distinct from generic agent persistence (state checkpoints) and from
  LongTermMemory (fact retrieval). Use when the user asks to "save chat history",
  "persist conversations", "resume a chat by session", "store chat in Postgres / DynamoDB".
---

# Persist Chat History Skill

Process steps in order. Do not skip ahead.

## Step 1 — Pick the Right Persistence Layer

Three Koog persistence concepts — easy to confuse:

- **Chat history persistence** (this skill) — `chat-history-jdbc`, `chat-history-aws`, `chat-memory-sql`. Stores the conversation messages keyed by session ID. Used to resume a chat where the user left off
- **Generic agent persistence** — checkpoints agent run state (node position, storage, planner state). Used to survive crashes mid-run; covered by `Skill(skill: "add-persistence")`
- **LongTermMemory** — stores extracted *facts* about the user/session for retrieval, not the raw messages; covered by `Skill(skill: "manage-state")`

If the user wants "user picks up the conversation tomorrow", this skill. If "agent resumes a crashed run from where it stopped", invoke `Skill(skill: "add-persistence")`. If "agent remembers things across sessions for retrieval", invoke `Skill(skill: "manage-state")`.

Proceed immediately to Step 2.

## Step 2 — Pick a Backend

Ask the user:

- **JDBC** (`chat-history-jdbc`) — any JDBC-compatible database (Postgres, MySQL, etc.). Most common
- **AWS** (`chat-history-aws`) — DynamoDB or AWS-hosted alternatives. For AWS-native deployments
- **SQL-typed chat memory** (`chat-memory-sql`) — SQL backend with stronger typing over chat messages. Use when message shape varies (attachments, tool calls) and you want a typed schema

Proceed immediately to Step 3.

## Step 3 — Add the Dependency

```kotlin
implementation("ai.koog:agents-features-chat-history-jdbc:1.0.0")
// or:
// implementation("ai.koog:agents-features-chat-history-aws:1.0.0")
// or:
// implementation("ai.koog:agents-features-chat-memory-sql:1.0.0")
```

For JDBC, also include the driver for the actual database (Postgres, etc.) — that's not provided by Koog.

Proceed immediately to Step 4.

## Step 4 — Install the Feature

Install in the `AIAgent(...)` trailing lambda. JDBC example:

```kotlin
import ai.koog.agents.features.chathistory.jdbc.JdbcChatHistory

val agent = AIAgent(
    promptExecutor = ...,
    llmModel = ...,
    systemPrompt = "...",
) {
    install(JdbcChatHistory) {
        jdbcUrl = System.getenv("DB_URL")
        username = System.getenv("DB_USER")
        password = System.getenv("DB_PASSWORD")
        // table name + schema — defaults are sensible, override if you must
    }
}
```

Always read credentials from environment variables, never inline the values.

Proceed immediately to Step 5.

## Step 5 — Resume by Session ID

`agent.run(input, sessionId = "...")` accepts a session ID. With chat-history persistence installed:

- A fresh session ID starts a new conversation; the feature writes messages as they happen
- An existing session ID loads prior messages from the backend, then continues — the LLM sees the full history without you having to thread it through

```kotlin
// Day 1
val sessionId = UUID.randomUUID().toString()
agent.run("Hello, I need help triaging a bug", sessionId = sessionId)
// store sessionId somewhere persistent (database, cookie, etc.)

// Day 2
val resumedSessionId = loadSessionIdForUser(...)
agent.run("Following up on the bug from yesterday", sessionId = resumedSessionId)
// agent sees both messages
```

Don't store the session ID inside the agent or in `AIAgentStorage` — that storage is run-scoped. The session ID is the user's identity in the chat history backend; persist it in your application's user/session layer.

Finish here.
