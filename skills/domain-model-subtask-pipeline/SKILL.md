---
name: domain-model-subtask-pipeline
description: >
  Author a Koog 1.0 agent as a typed pipeline of domain-modeled subtasks — tools
  sliced by access pattern (read / write / communication) into separate ToolSets,
  inter-subtask handoffs as `@Serializable` `@LLMDescription`-annotated data
  classes (not text prompts), each subtask wired with `subgraphWithTask<In, Out>`
  using its own model and tool subset, self-correction loops via
  `subgraphWithVerification<T>` + `CriticResult<T>`. The integration pattern
  Koog's own banking demo uses. Use when the user asks to "model the agent as a
  pipeline", "build a multi-stage agent with typed handoffs", "give each stage
  its own tools", "build a verify-and-fix loop with typed data", or describes a
  workflow with distinct phases that should hand structured data to each other.
---

# Domain Model Subtask Pipeline Skill

Process steps in order. Do not skip ahead.

Worked Kotlin example excerpts for the seven steps below (illustrative —
contains `...` placeholders for tool bodies and prompt-executor wiring; not a
standalone buildable file):

```text
skills/domain-model-subtask-pipeline/references/banking-example.md
```

## Step 1 — Confirm the Shape Fits

The pattern earns its complexity when the work has **distinct phases** that
produce structurally different intermediate artifacts. Examples: issue triage
(raw text → classified issue → fix proposal → verified fix); customer support
(question → identified problem → applied resolution → verified resolution);
code review (PR diff → categorized changes → review comments → applied edits).

Don't reach for this when the work is one-shot text-in-text-out — that's the
default `singleRunStrategy()`. Don't reach for it when the topology depends on
runtime findings — invoke `Skill(skill: "use-planner")` instead.

Proceed immediately to Step 2.

## Step 2 — Slice Tools by Access Pattern

The slicing axis is **access**, not feature. Group tools into separate `ToolSet`
classes by what they can do:

- **Communication tools** — ask the user, send email, post comments, request approval
- **Read tools** — query, list, search, get details (no side effects)
- **Write tools** — mutate, create, delete, transfer (real consequences)

**Constructor parameters are dependency injection — they are NOT visible to the
LLM.** `userId`, `sessionId`, database connections, HTTP clients go through the
constructor; the LLM only sees `@Tool`-annotated methods. This keeps
authorization context out of the LLM-controlled surface.

For tool registration mechanics, invoke `Skill(skill: "add-tool")`. See the
reference file for a worked three-`ToolSet` example.

Proceed immediately to Step 3.

## Step 3 — Define Typed Handoff Contracts

Each subtask's input and output is a `@Serializable` Kotlin data class with
`@LLMDescription` on the class and on every property. The LLM produces typed
JSON matching the schema; downstream subtasks consume the typed value.

The class-level `@LLMDescription` describes the contract; the property-level
descriptions describe each field. Both are read by the LLM and inform the
schema it produces.

The compiler verifies the chain — if subtask A's output type doesn't match
subtask B's input type, the strategy doesn't compile.

For top-level structured output (no subtask pipeline), invoke
`Skill(skill: "add-structured-output")`. See the reference file for worked
handoff classes.

Proceed immediately to Step 4.

## Step 4 — Build Each Subtask with `subgraphWithTask<In, Out>`

Each subtask is a `subgraphWithTask` (or `subgraphWithVerification` — see
Step 5) typed end-to-end. Pass the subset of tools the subtask needs plus the
model that fits the subtask's complexity:

```kotlin
val identifyProblem by subgraphWithTask<String, AccountIssueSummary>(
    tools = communicationTools.asTools() + readTools.asTools(),
    llmModel = OpenAIModels.Chat.GPT5_2,
) { input -> "Identify the problem:\n$input" }
```

**Match the model to the subtask:**

- Cheap classification / extraction → fast small models (GPT-5 mini, Haiku, Flash)
- Action / generation → mid-tier (Sonnet, GPT-5)
- Verification / reasoning → reasoning tier (O3, Opus, GPT-5 Pro)

The cost of running every subtask at the most-expensive tier compounds fast on
long chains. Mixed-model pipelines often cost a fraction of single-model ones
at similar output quality.

Pull `ai.koog:agents-ext:1.0.0` for `subgraphWithTask` /
`subgraphWithVerification`.

For graph DSL mechanics — edges, node naming, the rest of the surface — invoke
`Skill(skill: "author-strategy")`.

Proceed immediately to Step 5.

## Step 5 — Add a Verify-and-Adjust Loop with `subgraphWithVerification`

The headline benefit of the pattern is iterative self-correction without a
planner. `subgraphWithVerification<T>` produces a `CriticResult<T>` with
`.successful: Boolean`, `.feedback: String?`, `.input: T`. Branch on it:

```kotlin
edge(verifySolution forwardTo nodeFinish onCondition { it.successful } transformed { it.input })
edge(verifySolution forwardTo adjustSolution onCondition { !it.successful } transformed { it.feedback.orEmpty() })
edge(adjustSolution forwardTo verifySolution)
```

`transformed { it.input }` pulls the verified payload out of `CriticResult<T>`
on the success edge. `transformed { it.feedback.orEmpty() }` coerces the
nullable critic feedback to a non-null `String` for the adjust subgraph's
input. The adjust→verify back-edge closes the loop.

**The adjust subgraph re-runs the action.** It needs the same write access as
the action phase — communication-only adjustment cannot apply the corrected
fix. Grant adjust the read + write `ToolSet`s.

**Cap the loop.** Without a counter, a stubborn critic and a stubborn adjuster
can ping-pong forever within `maxIterations`. Track attempts in
`AIAgentStorage` (see `Skill(skill: "manage-state")`) and add an edge from
`adjustSolution` to `nodeFinish` when attempts exceed a sensible bound.

Proceed immediately to Step 6.

## Step 6 — Trust the Auto-Shared Message History

Koog shares the message history across subtasks automatically, even when each
subtask uses a different model. You do NOT thread the history manually between
subgraphs. Tool calls from `identifyProblem` are visible to `fixProblem`'s
LLM; `fixProblem`'s actions are visible to `verifySolution`.

The implication: if the chain is long, history grows. Compress at deliberate
boundaries — typically after the verification loop converges, or between
unrelated phases — inside a write session: `llm.writeSession {
replaceHistoryWithTLDR() }`. Pass a `HistoryCompressionStrategy` variant
when the default TL;DR shape doesn't fit.

For history-compression mechanics and the full set of
`HistoryCompressionStrategy` variants, invoke `Skill(skill: "manage-state")`.

Proceed immediately to Step 7.

## Step 7 — Wire the Strategy into `AIAgent`

Pass the strategy explicitly as `strategy =` (it's not the default). Register
every tool at the agent level via `toolRegistry`; each subgraph then selects
its subset via `subgraphWithTask`'s `tools =` parameter — the agent's registry
is one flat namespace, so don't try to register tools per-subgraph.

Set `maxIterations` higher than the default 50 — pipelines chain LLM calls and
the default runs out fast.

Run `./gradlew build`. The most common failure is a type mismatch on an edge
predicate — the chain only compiles when every subgraph's output type matches
the next subgraph's input type. Go back to Step 3 if types don't line up.

Full agent-construction snippet:

```text
skills/domain-model-subtask-pipeline/references/banking-example.md
```

Finish here.
