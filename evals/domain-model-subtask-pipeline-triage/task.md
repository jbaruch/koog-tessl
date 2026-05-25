# Build a Support Triage Agent With Distinct Phases

## Problem/Feature Description

A developer is building an agent for a customer-support team. They describe the workflow as four phases: first the agent should understand what the customer's issue actually is from their message, then take whatever action resolves it, then check whether the resolution actually worked, and if it didn't — adjust and re-check until it does.

They say the phases have very different shapes:

- The understanding phase looks at the message, asks clarifying questions, and reads the customer's account history. It shouldn't be able to change anything yet
- The action phase reads account state and applies changes (refunds, transfers, dispute openings). It shouldn't be talking to the customer at this point
- The verification phase looks at the result and may follow up with the customer to confirm. It shouldn't be making any further changes
- The adjustment phase re-runs the action with whatever the verifier flagged

They also mention: the understanding phase doesn't need the smartest model — a cheap one is fine. The action phase needs a mid-tier model. The verifier should be the most expensive reasoning model. They want each phase to produce a well-defined intermediate result that the next phase consumes — not a free-form text handoff.

## Output Specification

Walk through how to structure this in Koog. Produce the agent code as a single Kotlin file along with the necessary Gradle dependency lines, labeled.
