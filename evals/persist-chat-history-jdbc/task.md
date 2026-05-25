# Let Users Resume Conversations Across Days

## Problem/Feature Description

A developer is building a customer-support chatbot powered by a Koog 1.0 agent. Today, every page reload starts a fresh conversation — the user has to re-explain their context. The developer wants conversations to persist by user account, so a user who returns tomorrow continues where they left off.

They have a Postgres database available; the JDBC URL and credentials are in environment variables (`DB_URL`, `DB_USER`, `DB_PASSWORD`). They mentioned: "Each authenticated user has a userId — that's the session boundary for me."

## Output Specification

Walk through what to install and how to wire it into the agent's invocation pattern. Produce the modified agent construction and the route/handler code as a single response, labeled.
