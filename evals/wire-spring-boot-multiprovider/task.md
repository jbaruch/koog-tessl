# Wire a Spring Boot App With Two LLM Providers

## Problem/Feature Description

A developer maintains a Spring Boot service that currently uses OpenAI for some tasks. They want to keep OpenAI for the existing functionality and add Google as a second provider for a new feature — they'll route between models depending on the task at runtime. Both providers' API keys are in their environment.

The service code currently injects an `LLMClient` bean directly into one of their services. They mention that's how the previous engineer wrote it.

## Output Specification

Walk through what changes are needed to add Google as a second provider, both in configuration and in service code. Produce the modified `application.yml` and the relevant service code as a single response, labeled.
