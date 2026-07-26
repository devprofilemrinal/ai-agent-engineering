# Phase 01 Project — Token & Cost Estimator CLI

## Scenario

Before your team commits to an agent design, someone has to answer "roughly what will this cost per month at expected volume, and where are we at risk of hitting context limits?" — the same due-diligence question you'd expect before sizing a database or choosing an EC2 instance type. This project builds the tool that answers it: a small, dependency-free Java command-line utility that estimates token count and dollar cost for a piece of text, without ever calling a real LLM API.

## Functional requirements

1. Accept a text file path (or stdin) containing a prompt, and output an estimated token count using the character-based heuristics from `01-tokens-and-tokenization.md` (roughly 4 characters per token for prose, 3 for code/structured text).
2. Let the user specify whether the input is "prose" or "code," and apply the corresponding estimation ratio.
3. Accept a configurable expected output length (in estimated tokens) as a separate input, since input and output are priced differently.
4. Accept configurable input and output token rates (price per million tokens) so the tool isn't hardcoded to one provider's current pricing.
5. Output a total estimated cost for a single call, using the formula from `06-cost-and-latency-mechanics.md`.
6. Accept a "calls per month" figure and output a projected monthly cost.
7. Warn (do not silently proceed) if the combined input + expected-output token estimate would exceed a configurable context window size.

## Constraints

- No external HTTP calls, no API keys, no network access at all — this tool only estimates, it never verifies against a real tokenizer or live API.
- No third-party tokenization libraries — the estimation must use the simple character-based heuristics from Phase 01, deliberately, so you feel their limitations directly (you'll notice they're approximate, which is the point).
- Must run as a plain Java CLI (a `main` method invoked via `java`), not a Spring Boot application — this project is about the mechanics, not the framework, and frameworks haven't been introduced yet.

## What "done" looks like

- Running the tool against a short prose file and a short code file produces two different token estimates for texts of similar character length, demonstrating the prose/code ratio difference from file 1.
- Running the tool with a small `calls per month` value and a large one shows the monthly cost scaling linearly, demonstrating the compounding cost behavior from file 6.
- Deliberately feeding in a very large file plus a large expected-output value triggers the context-window warning rather than silently computing a number.
- You can explain, for any number the tool prints, which specific mechanic from this phase's files produced it.

## Extension

Once Phase 02 gives you a real API client, extend this tool to optionally call the provider's actual token-counting capability (if available) and print both the heuristic estimate and the real count side by side — this turns the tool into a way of calibrating how far off your heuristic estimates actually are for your team's typical prompts.