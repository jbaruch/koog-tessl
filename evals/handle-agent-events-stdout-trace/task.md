# Print a Live Trace of Tool Calls During a Demo

## Problem/Feature Description

A developer is preparing a conference demo of a Koog 1.0 agent. During the live demo they want the audience to see each tool call as it happens — a one-line print to stdout for every tool invocation, showing the tool name and arguments, with an arrow indicating start vs end. They don't want full production observability for this — no Langfuse, no OTel collector — just human-readable stdout output for the projector.

The agent is otherwise unchanged from a basic `AIAgent(...)` call.

## Output Specification

Walk through what to add. Produce the modified agent construction and Gradle dependency change as a single response, labeled.
