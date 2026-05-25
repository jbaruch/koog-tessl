# Author a Generate → Verify → Fix Strategy

## Problem/Feature Description

A developer is building an agent that drafts API documentation from code. They want a multi-step workflow where the agent:

1. Generates a first draft of the documentation
2. Has a separate verifier look it over for accuracy issues against the source code
3. If problems are found, sends them to a fixer that revises the draft using the feedback
4. Loops between verify and fix until the verifier is satisfied
5. Returns the final draft

Each phase needs to use its own set of tools — the generator has file-read tools, the verifier has read + grep, the fixer has read + edit + grep. The developer mentions they want the verifier to use a cheaper model than the fixer.

## Output Specification

Walk through how to express this as a Koog strategy. Produce the strategy source as a single Kotlin file, labeled.
