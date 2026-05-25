---
name: use-planner
description: >
  Pick and wire a planner-driven Koog 1.0 agent — either LLM-based (the LLM picks the
  next action each turn, optionally with a critic loop) or GOAP (a classical planner
  searches a typed state space toward a goal). Pulls `ai.koog:agents-planner`, constructs
  the planner strategy, and wires it into `AIAgent(...)`. Use when the user asks to
  "use a planner", "let the agent plan", "use GOAP", "build a planning agent",
  names any of `Planners.llmBased`, `Planners.llmBasedWithCritic`, `Planners.goap`,
  `PlannerAIAgent`, `agents-planner`, or describes an open-ended task whose step
  sequence depends on runtime context.
---

# Use Planner Skill

This skill is an action router — pick the step that matches the user's intent and execute only that step. Do not run other steps; do not parallelize.

Available actions:

- **Step 1** — Confirm the planner is the right primitive. If not, redirect via `Skill(skill: "author-strategy")`. If yes, the user picks LLM-based or GOAP and the relevant Step (2 or 3) follows
- **Step 2** — LLM-based planner (`Planners.llmBased` or `Planners.llmBasedWithCritic`)
- **Step 3** — GOAP planner (`Planners.goap`)

## Step 1 — Confirm the Planner Fits

The graph DSL (`strategy { ... }`) is the right default. A planner is the right choice when **any** of these are true:

- The action ordering genuinely depends on runtime findings (the next step depends on what the LLM saw in the last one) — not just on which tool the LLM picked
- The action space is large enough that hardcoding edges would be a maintenance burden
- You're willing to trade extra LLM round-trips for autonomy

If the user's description matches "I know the topology, the LLM just picks tools within nodes", redirect — they don't need a planner, they need a graph:

- Invoke `Skill(skill: "author-strategy")` and run it end-to-end through its Step 8
- Write the graph DSL code to disk per `author-strategy`'s Step 8
- Add a top-of-file comment in the produced strategy file naming the topology as the disqualifying signal for a planner
- Add a second top-of-file comment naming the extra LLM round-trips a planner would have added

Then pick the planner variant from the user's description without blocking on a clarifying question:

- LLM-based (default) — pick when ordering depends on runtime findings and state is unstructured prose
- GOAP — pick only when the user names "GOAP", "classical planner", "state space search", or supplies a typed `data class` state and precondition/effect pairs
- `Planners.llmBasedWithCritic` — pick when the user names "critic", "verify", or asks for output-quality grading

Proceed to Step 2 for LLM-based / critic, Step 3 for GOAP.

## Step 2 — LLM-Based Planner

Add the planner module to `build.gradle.kts`:

```kotlin
implementation("ai.koog:agents-planner:1.0.0")
```

Construct the planner strategy and wire into `AIAgent(...)`:

```kotlin
import ai.koog.agents.planner.Planners
import ai.koog.agents.core.agent.AIAgent

val strategy = Planners.llmBased("myStrategy")
// or with a critic loop for output quality:
// val strategy = Planners.llmBasedWithCritic("myStrategy")

val agent = AIAgent(
    promptExecutor = ...,
    llmModel = ...,
    toolRegistry = ...,
    systemPrompt = "...",
    strategy = strategy,
    maxIterations = 400,  // planners chain many LLM steps — default 50 is far too low
)
```

The critic variant adds an inner verify loop where a second LLM call grades each planner step before it's accepted. Use when output quality matters more than throughput; expect roughly 2× the LLM cost.

`Planners.llmBased(name)` is the new constructor — the old `AIAgentPlannerStrategy.builder()` is gone in 1.0 (#1997).

Finish here.

## Step 3 — GOAP Planner

GOAP needs a **typed, serializable state class** and a set of actions defined as `(precondition, effect)` pairs. If the state cannot be expressed as a `data class` with a finite set of fields, GOAP is the wrong tool — use Step 2.

Add the planner module:

```kotlin
implementation("ai.koog:agents-planner:1.0.0")
```

Define the state and the planner:

```kotlin
import ai.koog.agents.planner.Planners
import kotlinx.serialization.Serializable

@Serializable
data class MyState(
    val hasItem: Boolean = false,
    val location: String = "start",
)

val strategy = Planners.goap("myStrategy", ::MyState) {
    action(
        name = "pickup",
        precondition = { s -> !s.hasItem && s.location == "warehouse" },
        belief = { s -> s.copy(hasItem = true) }
    ) { _, s ->
        // actual side-effect during execution
        s.copy(hasItem = true)
    }

    action(
        name = "move_to_warehouse",
        precondition = { s -> s.location != "warehouse" },
        belief = { s -> s.copy(location = "warehouse") }
    ) { _, s -> s.copy(location = "warehouse") }

    goal("haveItem", condition = { s -> s.hasItem })
}
```

`belief` is the planner's *predicted* effect (used during planning); the action body is the *actual* effect at execution time. They usually match but can diverge — e.g., the actual effect may also update non-state-tracked fields.

Wire into the agent:

```kotlin
val agent = AIAgent(
    promptExecutor = ...,
    llmModel = ...,
    toolRegistry = ...,
    systemPrompt = "...",
    strategy = strategy,
    maxIterations = 100,
)
```

Finish here.
