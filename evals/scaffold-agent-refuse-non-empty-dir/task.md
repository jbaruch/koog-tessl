# Scaffold Into a Directory That Already Contains Files

## Problem/Feature Description

A developer points the scaffold skill at `~/work/old-experiment`. That directory already exists and contains several files from a previous attempt at building a different project — there's a `README.md`, some Kotlin source under `src/`, and a half-written `build.gradle.kts`. The developer has not told the skill what to do about the existing content; they just supplied the path and a provider choice.

## Output Specification

Walk through what the skill should do in this situation. Capture your reasoning and the exact sequence of actions (or refusals) in a file named `scaffold-plan.md`.
