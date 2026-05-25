# Build a Triage Agent Whose Steps Depend on Findings

## Problem/Feature Description

A developer is building a customer-support triage agent. The agent receives an open ticket and must decide what to do — but the right next step depends on what it finds:

- If the ticket text mentions a recent outage, look up the outage status before responding
- If the ticket mentions an account, look up the account's tier before responding
- If neither, just classify and respond
- After any look-up, decide whether to escalate based on what the look-up returned

The developer says: "I can't pre-wire this — the path through the steps depends on what the ticket says and what the look-ups return." They want the model to make those decisions itself, given the available tools.

They're prepared to pay for the extra LLM round-trips. They asked for whichever Koog primitive matches this shape.

## Output Specification

Walk through which Koog primitive applies and produce the agent construction code as a single Kotlin file, labeled.
