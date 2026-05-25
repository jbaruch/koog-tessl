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

Process steps in order. Do not skip ahead. This skill is the integration of
several lower-level concerns; per-step mechanics that go beyond what fits here
are covered in the linked skills.

## Step 1 — Confirm the Shape Fits

The pipeline pattern earns its complexity when the work has **distinct phases**
that produce structurally different intermediate artifacts. Examples:

- Issue triage: raw text → classified issue → reproduction attempt → fix proposal → verified fix
- Customer support: question → identified problem → applied resolution → verified resolution
- Code review: PR diff → categorized changes → review comments → applied edits

Don't reach for this when the work is one-shot text-in-text-out — that's the
default `singleRunStrategy()`. Don't reach for it when the topology depends on
runtime findings — invoke `Skill(skill: "use-planner")` instead.

Proceed immediately to Step 2.

## Step 2 — Slice Tools by Access Pattern

The slicing axis is **access**, not feature. Group tools into separate `ToolSet`
classes by what they can do, not by what entity they touch:

- **Communication tools** — ask the user, send email, post comments, request approval
- **Read tools** — query data, list, search, get details (no side effects)
- **Write tools** — mutate, create, delete, transfer (real consequences)

```kotlin
class CommunicationTools(private val sessionId: String) : ToolSet {
    @Tool
    @LLMDescription("Ask the user a clarifying question and wait for the reply")
    fun askUser(@LLMDescription("Question text") question: String): String { ... }
}

class AccountReadTools(private val userId: String) : ToolSet {
    @Tool
    @LLMDescription("Get account balance (in USD) for the current user")
    fun getAccountBalance(): Int { ... }

    @Tool
    @LLMDescription("Returns a list of transactions for the current user")
    fun getLatestTransactions(startDate: Instant?, status: Transaction.Status?): List<Transaction> { ... }
}

class AccountWriteTools(private val userId: String) : ToolSet {
    @Tool
    @LLMDescription("Transfers money to the recipient")
    fun transferMoney(
        @LLMDescription("ID of the recipient") recipientId: String,
        @LLMDescription("Amount in USD to be transfered") amount: Int,
    ): TransferResult { ... }
}
```

**Constructor parameters are dependency injection — they are NOT visible to the
LLM.** The `userId`, `sessionId`, database connections, HTTP clients all go
through the constructor; the LLM only sees `@Tool`-annotated methods. This is
how you keep authorization context out of the LLM-controlled surface.

For tool registration mechanics that go beyond this, invoke
`Skill(skill: "add-tool")`.

Proceed immediately to Step 3.

## Step 3 — Define Typed Handoff Contracts

Each subtask's input and output is a `@Serializable` Kotlin data class with
`@LLMDescription` on every property. The LLM produces typed JSON matching the
schema; downstream subtasks consume the typed value:

```kotlin
@LLMDescription("Full info about the user's issue with the bank account")
@Serializable
data class AccountIssueSummary(
    @property:LLMDescription("Account number of the user in the database")
    val accountNumber: String,

    @property:LLMDescription("Username of the account holder")
    val username: String,

    @property:LLMDescription("Current account balance in US dollars")
    val currentBalance: Int,

    @property:LLMDescription("ID of the transaction related to this issue, if applicable")
    val relatedTransactionId: String?,

    @property:LLMDescription("What exactly is the user's issue with their account or transaction")
    val problem: String,

    @property:LLMDescription("Was the issue already resolved?")
    val resolved: Boolean,
)

@LLMDescription("Summary about what was done to resolve the issue")
@Serializable
data class AccountIssueSolution(
    @property:LLMDescription("Account number that was affected") val accountNumber: String,
    @property:LLMDescription("Brief summary of the actions taken to resolve the issue") val actionsTaken: String,
)
```

The class-level `@LLMDescription` describes the contract; the property-level
descriptions describe each field. Both are read by the LLM and inform the schema
it produces.

This is the contract: *naive prompting gives you hope; domain modeling gives a
contract.* The compiler verifies the chain — if subtask A's output type doesn't
match subtask B's input type, the strategy doesn't compile.

For top-level structured output (no subtask pipeline), invoke
`Skill(skill: "add-structured-output")`.

Proceed immediately to Step 4.

## Step 4 — Build Each Subtask with `subgraphWithTask<In, Out>`

Each subtask is a `subgraphWithTask` (or `subgraphWithVerification` — see
Step 5) typed end-to-end. Pass the subset of tools the subtask needs plus the
model that fits the subtask's complexity:

```kotlin
val identifyProblem by subgraphWithTask<String, AccountIssueSummary>(
    tools = communicationTools.asTools() + readTools.asTools(),
    llmModel = OpenAIModels.Chat.GPT5_2,   // cheap classification
) { input -> "Identify the problem, formulate a problem description:\n$input" }

val fixProblem by subgraphWithTask<AccountIssueSummary, AccountIssueSolution>(
    tools = readTools.asTools() + writeTools.asTools(),
    llmModel = AnthropicModels.Sonnet_4,   // smart action
) { description -> "Now solve the user's problem:\n$description" }
```

**Match the model to the subtask:**

- Cheap classification / extraction → fast small models (GPT-5 mini, Haiku, Flash)
- Action / generation → mid-tier (Sonnet, GPT-5)
- Verification / reasoning → reasoning tier (O3, Opus, GPT-5 Pro)

The cost of running every subtask at the most-expensive tier compounds fast on
long chains. Mixed-model pipelines often cost a fraction of single-model ones at
similar output quality.

Pull `ai.koog:agents-ext:1.0.0+` for `subgraphWithTask` / `subgraphWithVerification`.

For graph DSL mechanics — edges, node naming, the rest of the surface — invoke
`Skill(skill: "author-strategy")`.

Proceed immediately to Step 5.

## Step 5 — Add a Verify-and-Adjust Loop with `subgraphWithVerification`

The headline benefit of the pattern is iterative self-correction without a
planner. `subgraphWithVerification<T>` produces a `CriticResult<T>` with
`.successful: Boolean`, `.feedback: String?`, `.input: T`. Branch on it:

```kotlin
val verifySolution by subgraphWithVerification<AccountIssueSolution>(
    tools = communicationTools.asTools() + readTools.asTools(),
    llmModel = OpenAIModels.Chat.O3,
) { solution -> "Now verify that the problem is actually solved:\n$solution" }

val adjustSolution by subgraphWithTask<String, AccountIssueSolution>(
    tools = communicationTools.asTools() + readTools.asTools(),
    llmModel = AnthropicModels.Sonnet_4,
) { feedback -> "Adjust the solution using this feedback:\n$feedback" }

edge(nodeStart forwardTo identifyProblem)
edge(identifyProblem forwardTo fixProblem)
edge(fixProblem forwardTo verifySolution)
edge(verifySolution forwardTo nodeFinish onCondition { it.successful } transformed { it.input })
edge(verifySolution forwardTo adjustSolution onCondition { !it.successful } transformed { it.feedback })
edge(adjustSolution forwardTo verifySolution)
```

The `transformed { it.input }` pulls the verified payload out of
`CriticResult<T>` for the success edge; `transformed { it.feedback }` pulls the
critic's feedback for the failure edge. The adjust→verify back-edge closes the
loop.

**Cap the loop.** Without a counter, a stubborn critic and a stubborn adjuster
can ping-pong forever within `maxIterations`. Track attempts in
`AIAgentStorage` (see `Skill(skill: "manage-state")`) and have an edge from
`adjustSolution` to `nodeFinish` when attempts exceed a sensible bound.

Proceed immediately to Step 6.

## Step 6 — Trust the Auto-Shared Message History

Koog automatically shares the message history across subtasks, even when each
subtask uses a different model. You do NOT thread the history manually between
subgraphs. Tool calls from `identifyProblem` are visible to `fixProblem`'s LLM;
`fixProblem`'s actions are visible to `verifySolution`.

The implication: if the chain is long, history grows. Compress at deliberate
boundaries — typically after the verification loop converges, or between
unrelated phases. Use `nodeLLMCompressHistory<T>()` (the typed compression node,
which preserves the output type when compressing within a subtask boundary):

```kotlin
val compressHistory by nodeLLMCompressHistory<AccountIssueSolution>(
    strategy = HistoryCompressionStrategy.Chunked(chunkSize = 20)
)
```

For history-compression mechanics, invoke `Skill(skill: "manage-state")`.

Proceed immediately to Step 7.

## Step 7 — Wire the Strategy into `AIAgent`

The strategy is built; pass it explicitly as `strategy =` (it's not the
default):

```kotlin
val triageStrategy = strategy<String, AccountIssueSolution>("issue-triage") {
    // val identifyProblem by ... (Step 4)
    // val fixProblem by ... (Step 4)
    // val verifySolution by ... (Step 5)
    // val adjustSolution by ... (Step 5)
    // edges as in Step 5
}

val agent = AIAgent(
    promptExecutor = ...,
    llmModel = OpenAIModels.Chat.GPT5_2,    // default model — subtasks override
    toolRegistry = ToolRegistry {
        tools(communicationTools.asTools())
        tools(readTools.asTools())
        tools(writeTools.asTools())
    },
    systemPrompt = "...",
    strategy = triageStrategy,
    maxIterations = 200,        // pipelines chain LLM calls — default 50 is too low
)
```

The agent-level `toolRegistry` registers every tool; each subgraph then selects
its subset via `subgraphWithTask`'s `tools =` parameter. Don't try to register
tools per-subgraph at the agent level — the agent's registry is one flat
namespace.

Run `./gradlew build`. The most common failure is a type mismatch on an edge
predicate — the chain only compiles when every subgraph's output type matches
the next subgraph's input type. Go back to Step 3 if types don't line up.

Finish here.
