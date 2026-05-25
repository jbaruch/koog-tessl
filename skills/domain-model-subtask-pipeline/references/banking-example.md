# Banking Pipeline — Full Example

End-to-end Kotlin file matching the seven-step skill. Adapted from the
JetBrains KotlinConf 2026 banking demo — four typed phases (`identifyProblem`
→ `fixProblem` → `verifySolution` ⇄ `adjustSolution`), tools sliced by access
into three `ToolSet`s, mixed per-phase models.

## Tools sliced by access pattern

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
        @LLMDescription("Amount in USD to be transferred") amount: Int,
    ): TransferResult { ... }
}
```

## Typed handoff classes

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

## Strategy: four subgraphs + verify/adjust loop

```kotlin
val triageStrategy = strategy<String, AccountIssueSolution>("issue-triage") {
    val identifyProblem by subgraphWithTask<String, AccountIssueSummary>(
        tools = communicationTools.asTools() + readTools.asTools(),
        llmModel = OpenAIModels.Chat.GPT5_2,   // cheap classification
    ) { input -> "Identify the problem, formulate a problem description:\n$input" }

    val fixProblem by subgraphWithTask<AccountIssueSummary, AccountIssueSolution>(
        tools = readTools.asTools() + writeTools.asTools(),
        llmModel = AnthropicModels.Sonnet_4,   // smart action
    ) { description -> "Now solve the user's problem:\n$description" }

    val verifySolution by subgraphWithVerification<AccountIssueSolution>(
        tools = communicationTools.asTools() + readTools.asTools(),
        llmModel = OpenAIModels.Chat.O3,       // reasoning verification
    ) { solution -> "Now verify that the problem is actually solved:\n$solution" }

    val adjustSolution by subgraphWithTask<String, AccountIssueSolution>(
        tools = readTools.asTools() + writeTools.asTools(),   // re-runs the action — needs write
        llmModel = AnthropicModels.Sonnet_4,
    ) { feedback -> "Adjust the solution using this feedback:\n$feedback" }

    edge(nodeStart forwardTo identifyProblem)
    edge(identifyProblem forwardTo fixProblem)
    edge(fixProblem forwardTo verifySolution)
    edge(verifySolution forwardTo nodeFinish onCondition { it.successful } transformed { it.input })
    edge(verifySolution forwardTo adjustSolution onCondition { !it.successful } transformed { it.feedback.orEmpty() })
    edge(adjustSolution forwardTo verifySolution)
}
```

The `transformed { it.input }` pulls the verified payload out of
`CriticResult<T>` on the success edge. The failure edge coerces
`it.feedback` (nullable `String?`) to non-null via `.orEmpty()` so it matches
`adjustSolution`'s `subgraphWithTask<String, _>` input type.

## Agent construction

```kotlin
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

## History compression between phases

```kotlin
import ai.koog.agents.core.dsl.extension.HistoryCompressionStrategy

// inside a node body, between phases
llm.writeSession {
    replaceHistoryWithTLDR()   // collapses everything into a single TL;DR summary
}
```

Invoke at the end of a phase or the start of the next, not at every node; see
`Skill(skill: "manage-state")` for `HistoryCompressionStrategy` variants
(`Chunked`, `FromLastNMessages`, `FactRetrieval`, etc.).
