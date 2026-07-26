# Phase 05 — Tool Use and Function Calling

## Why this phase exists

Phase 02, file 5 already showed you the wire protocol for tool calling — the JSON shape of `tools`, `tool_use`, and `tool_result`. This phase is where that protocol becomes an engineering discipline: designing tools the model can use reliably, understanding why and when it decides to call one, validating what it asks for before executing anything, and deciding when a human needs to approve an action before it happens. This is the single most important prerequisite for everything called an "agent" in this handbook — a model that can only produce text is a chatbot; a model that can request real actions, safely, is the seed of an agent.

## What this phase covers

1. `01-designing-tool-schemas.md` — how to describe a tool so the model actually uses it correctly, and the specific mistakes that cause unreliable tool selection.
2. `02-how-models-decide-to-call-tools.md` — the mechanism behind tool selection, tying back to Phase 01's sampling and attention concepts.
3. `03-validating-and-sandboxing-tool-execution.md` — treating every tool-call request as untrusted input, the way you'd treat any external client request to a REST endpoint.
4. `04-human-in-the-loop-confirmation-patterns.md` — when and how to gate irreversible actions behind explicit human approval.
5. `05-tool-calling-in-langchain4j-and-spring-ai.md` — how both frameworks package tool definition and dispatch.
6. `06-project-java-devops-tool-agent.md` — a real tool-using agent against a Kubernetes-style API, combining every file in this phase.

## Prerequisites

Phase 02 (the wire protocol itself) and Phase 01 (sampling behavior, to understand why a model can request a tool with malformed or invented arguments).

## What you gain from this phase

The ability to safely let a model trigger real backend operations — not trusting its output blindly, but treating every tool-call request the way you'd treat any request arriving at a public API boundary: validated, scoped, and — where the action is consequential — gated behind confirmation. This is the exact discipline Phase 07's agent loops depend on; an agent loop that calls unvalidated tools is a loop that can take real, unintended actions repeatedly and automatically.