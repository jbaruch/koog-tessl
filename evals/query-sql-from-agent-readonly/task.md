# Let an Analytics Agent Read From the Events Table

## Problem/Feature Description

A developer is building an analytics-helper agent. Internal staff ask it natural-language questions like "how many signups did we have last week?" and the agent answers by querying their Postgres warehouse. The developer wants the agent to be able to query the database directly — but only the `public.events` and `public.users` tables, and read-only.

They have a JDBC URL and read-only credentials already provisioned at the database (`ANALYTICS_DB_URL`, `ANALYTICS_DB_USER`, `ANALYTICS_DB_PASSWORD`). They want to cap result sets at a reasonable size to prevent runaway queries.

## Output Specification

Walk through what to install and configure. Produce the modified agent construction and the system prompt section that pins the schema, as a single response, labeled.
