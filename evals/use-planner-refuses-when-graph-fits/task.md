# Build a "Planning" Agent for a Fully Known Workflow

## Problem/Feature Description

A developer says they want to "use a planner" for a customer-support agent. When asked to describe the workflow, they say: every conversation goes through the same three steps in the same order — first classify the ticket type, then look up the customer's account, then either escalate or reply. The LLM does pick which tool to call inside each step (there are a few classification tools, several lookup tools, several reply templates), but the order of the three phases never changes.

They explicitly say they like the idea of "letting the agent plan its work" because the term sounded right.

## Output Specification

Walk through what to build. Produce the response as a single message — including any code you'd write or any clarification you'd offer the developer.
