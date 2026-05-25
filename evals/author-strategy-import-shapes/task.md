# Add a Tool-Handling Loop to a Koog Strategy

## Problem/Feature Description

A developer is writing a Koog 1.0 strategy by hand (not using `singleRunStrategy()`). The agent needs to: take a user message, call the LLM, and from there one of two things happens — either the LLM produced text, in which case the agent finishes and returns the text; or the LLM tried to call tools, in which case those tools execute and their results go back to the LLM, looping until it produces text. Standard tool-handling shape.

They have a scaffolded `main()` that constructs an `AIAgent`, and they're about to write a custom strategy file. They are aware that 1.0 made the auto-writeback for tool results go away — they want the explicit `nodeExecuteTools` → `nodeLLMSendToolResults` chain.

## Output Specification

Produce the strategy file as Kotlin source, including the `import` block at the top.
