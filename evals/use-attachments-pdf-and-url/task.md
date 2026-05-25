# Contract and Product Image Analyzer

## Problem/Feature Description

A legal-tech startup is building a Koog 1.0 agent that helps procurement teams review vendor contracts alongside product specifications. Each review request pairs a PDF contract (stored on the local filesystem) with a publicly hosted product image URL. The agent must read both artifacts in the same LLM call and return a summary of any concerns found in the contract and whether the product image matches the described item.

The engineering team has chosen to use the Anthropic provider for this agent. They need you to write the core strategy that accepts the PDF path and image URL as inputs, wires them into a single user message, and sends it to the LLM. The PDF files can be large (tens of megabytes), and the product images are served from a CDN.

## Output Specification

Produce a single Kotlin file (`contract_analyzer.kt`) containing:
- The strategy implementation that accepts the PDF path and image URL as inputs
- A `main` function (or usage comment) that shows how to run the agent with a sample PDF path and image URL

The output file should compile cleanly (no missing imports, valid Kotlin syntax).
