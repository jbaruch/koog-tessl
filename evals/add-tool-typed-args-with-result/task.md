# Add a Database Query Tool with Typed Arguments

## Problem/Feature Description

A developer has a Koog 1.0 agent that helps customer-support staff look up account information. They want to add a tool that queries an internal accounts database. The query takes a structured input — they already have these data classes in their project:

```kotlin
@Serializable
data class AccountLookupRequest(
    val accountId: String,
    val region: String,
    val includeArchived: Boolean,
)

@Serializable
data class AccountLookupResult(
    val displayName: String,
    val tier: String,
    val openTickets: Int,
    val lastSeenIso: String,
)
```

They explicitly say they want the tool's input and output to remain these typed shapes — not a JSON blob, not a flattened String — because downstream code in their project also consumes the typed result.

They've already implemented the actual query logic in a `suspend fun queryAccount(req: AccountLookupRequest): AccountLookupResult` function.

## Output Specification

Walk through how to expose this typed query to the agent. Produce the modified or new source files in a single response, with each file labeled.
