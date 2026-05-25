# Scaffold an Agent into an Existing Empty Directory with Anthropic

## Problem/Feature Description

A developer has already created the target directory (`~/work/triage-bot`) — it exists but is completely empty. They want to use Anthropic as the LLM backend and Gradle (Kotlin DSL). Their Anthropic key is in their shell environment.

They invoke the scaffold skill, naming the directory and "Anthropic" as the provider.

## Output Specification

Walk through what the skill produces. Capture the resulting `build.gradle.kts` and `Main.kt` contents in a single response, with each file clearly labeled.
