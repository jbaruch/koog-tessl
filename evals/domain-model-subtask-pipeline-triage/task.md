# Build a Customer-Support Agent for a Bank

## Problem/Feature Description

A developer is building an agent for a bank's customer-support team. The agent handles complaints about specific transactions and account problems — disputed charges, missed transfers, balance discrepancies. End-to-end, the agent should: figure out what the customer is actually complaining about (possibly asking a clarifying question first), do whatever needs doing to resolve it (looking up the account, issuing refunds or transfers, opening dispute tickets), then check whether the issue is actually resolved — and if it isn't, try again instead of handing back to a human.

Two operational constraints the developer cares about:

- The agent shouldn't be able to issue refunds or transfers while it's still asking the customer clarifying questions, and it shouldn't be initiating chatty back-and-forth with the customer in the middle of executing a refund. The work is gated, not interleaved.
- Token cost matters. The developer doesn't want every step paying premium reasoning-model rates — only the step that genuinely needs reasoning should pay for it; the rest should use whatever's cheap and fast enough for that step's job.

## Output Specification

Walk through how to structure this in Koog.
