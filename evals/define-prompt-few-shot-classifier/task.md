# Build a Prompt With Few-Shot Examples

## Problem/Feature Description

A developer is building an agent that classifies short user feedback into categories (`bug`, `praise`, `question`, `noise`). They want to ground the model with a handful of labeled examples — they have these examples written down:

- "App keeps crashing on Settings → bug"
- "Love the new dark mode!! → praise"
- "How do I export to PDF? → question"
- "asdfgh → noise"

They're currently passing a single string to `systemPrompt = "..."` and finding that the model still misclassifies edge cases. They want to switch to a structured prompt with the examples wired in.

## Output Specification

Walk through how to express this with few-shot turns. Produce the prompt definition as a single Kotlin snippet, labeled.
