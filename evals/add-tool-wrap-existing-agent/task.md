# Wrap a Smaller Agent as a Tool for a Bigger One

## Problem/Feature Description

A developer is building a coding agent. The main agent edits files, runs tests, and opens PRs — it's expensive (Opus-class model, many tools). They've also built a smaller, cheaper agent whose only job is to find references to a symbol across a large codebase — it uses a Haiku-class model and just a couple of `grep`-style tools. They've already constructed both agents independently and both work on their own.

They want the main agent to be able to "ask the find agent" — that is, to invoke the find agent as if it were a single tool call, getting back the search result, then continue its work. They emphasize they don't want to inline the find logic into the main agent; the find agent should stay separate (different model, different system prompt, different tool set) for cost reasons.

## Output Specification

Walk through how to make the find agent callable from the main agent. Produce the modified source for the main agent's construction in a single response.
