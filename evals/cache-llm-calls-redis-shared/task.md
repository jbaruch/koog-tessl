# Share LLM Response Cache Across Worker Processes

## Problem/Feature Description

A developer runs a Koog 1.0 agent inside a pool of identical worker processes (4 instances behind a load balancer). They notice the same user inputs occasionally route to different workers and each one calls the LLM separately — duplicating cost. They want a cache layer that's shared across the workers so a response computed by one worker is reused by the others.

They already have a Redis instance available; the connection URL is in `REDIS_URL`. Their agent currently uses `simpleOpenAIExecutor`.

## Output Specification

Walk through what to install and how to wrap the executor. Produce the modified agent construction and dependency changes as a single response, labeled.
