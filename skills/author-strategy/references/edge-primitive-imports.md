# Edge Primitive Imports — Members vs Extensions

The infix vocabulary in the strategy DSL splits across two shapes. Inventing
a member import or omitting an extension import is the most common
copy-paste compile failure when authoring strategies by hand.

| Primitive | Kind | Import needed? |
|---|---|---|
| `forwardTo` | infix member on `AIAgentNodeBase` | No |
| `onCondition` | infix member on `AIAgentEdgeBuilderIntermediate` | No |
| `transformed` | infix member on `AIAgentEdgeBuilderIntermediate` | No |
| `onToolCalls` | top-level extension | `import ai.koog.agents.core.dsl.extension.onToolCalls` |
| `onTextMessage` | top-level extension | `import ai.koog.agents.core.dsl.extension.onTextMessage` |
| `onIsInstance` | top-level extension | `import ai.koog.agents.core.dsl.extension.onIsInstance` |
| `onMessageParts` | top-level extension | `import ai.koog.agents.core.dsl.extension.onMessageParts` |
| `onSuccessful` / `onFailure` | top-level extensions | `import ai.koog.agents.core.dsl.extension.onSuccessful` / `onFailure` |
| `asUserMessage` / `asToolResultMessage` | top-level extensions | `import ai.koog.agents.core.dsl.extension.asUserMessage` / `asToolResultMessage` |

Members travel with their receiver type and need no import. Extensions live
in `ai.koog.agents.core.dsl.extension.*` and must be imported by name.
Star-imports work but obscure which DSL surface is actually in use.
