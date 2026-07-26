# Phase 09 Project — Multi-Agent Research Pipeline

## Scenario

Phase 07 built a single research agent capable of searching, reading, and synthesizing an answer to a research question. This project asks you to build a *multi-agent* version of a similar capability — but only after first satisfying this phase's central, recurring demand: a concrete, specific justification for why this particular task genuinely benefits from multiple differentiated agents rather than one well-scoped agent from Phase 07. If you cannot produce that justification honestly, the correct outcome of this project is to conclude a single agent was sufficient, and to build a weaker case for multi-agent decomposition anyway would be a worse engineering outcome than an honest "this didn't need it."

## Functional requirements

1. **Write an explicit justification, before writing any code**, for why this task warrants a supervisor coordinating a search agent and a summarizer agent as distinct components, referencing file 1's decision diagram directly — what meaningfully different tool sets, reasoning styles, or system-prompt framings does each agent need that a single generalist agent would perform worse at combining?
2. **Implement the search agent and summarizer agent as separate, independently-scoped `ProductionAgentLoop` instances** (Phase 07, file 4), each with its own system prompt and tool set — the search agent should have search/fetch tools and a system prompt oriented around finding and identifying relevant sources; the summarizer agent should have no search tools at all and a system prompt oriented purely around synthesizing already-gathered material.
3. **Coordinate them through an explicit graph** (file 2), not just a flat supervisor delegation — include at minimum a conditional branch where, if the summarizer determines the gathered material is insufficient to answer the original question, control returns to the search agent for another round, rather than the summarizer being forced to produce a synthesis from inadequate material.
4. **Use a typed handoff contract** (file 4's `AgentHandoff` pattern) for every transition between the search and summarizer stages — no ad hoc string concatenation for what gets communicated at each handoff.
5. **Implement graph-level budget enforcement** (file 2's `maxTransitions`, plus each individual agent's own Phase 07 budget) — the system must halt cleanly and report a partial result if the graph exceeds a reasonable transition count, exactly mirroring Phase 07's halt-and-report behavior at the graph level rather than the single-agent level.
6. **Track and report cumulative cost across the entire multi-agent run** — every search agent invocation's cost and every summarizer agent invocation's cost, summed, not just the cost of whichever agent happened to run most recently, per file 1's discussion of cost propagating upward through nested agent invocations.

## Constraints

- The search agent must genuinely have no summarization-oriented instructions in its system prompt, and the summarizer agent must genuinely have no search tools registered — the separation of concerns must be real and enforced structurally (different tool registrations, different system prompts), not just implied by naming.
- The insufficient-material branch (search agent's insufficient-results conditional path back from the summarizer) must be demonstrated to actually trigger at least once during testing — a stress-test question deliberately chosen to require a second search round satisfies this requirement.
- The final written comparison (below) must include a genuine, specific measurement against the single-agent baseline from Phase 07 — not a general impression.

## What "done" looks like

- A written justification, produced before implementation, for why this task warrants multi-agent decomposition, referenced back to file 1's decision criteria.
- A working graph execution trace for at least one research question that required the insufficient-material branch to trigger — showing the summarizer's rejection of the first round of search results, the graph routing back to the search agent, a second search round, and a final, successful synthesis.
- A cost and iteration comparison: run the same research question through both this project's multi-agent pipeline and Phase 07's original single-agent research agent, and report the actual cost and total call count for each. State plainly whether the multi-agent version's result was measurably better, comparable, or worse than the single agent's, and whether that quality difference (if any) justified the cost difference you measured.
- A demonstrated graph-level budget halt, using a deliberately tight `maxTransitions` value on a question that would otherwise require more rounds than that budget allows.

## Extension

Having built this and measured it honestly against the Phase 07 baseline, write a short, candid retrospective: if you were advising a team building this for the first time, would you recommend they start with the single-agent version and only add multi-agent decomposition once they had concrete evidence (like the comparison you just produced) that it was needed — or does your own measured comparison suggest the multi-agent version was worth building from the start for this specific task? Defend whichever answer your own data actually supports, even if it's the less impressive-sounding one — this is the discipline this entire phase has been building toward, and Phase 10's evaluation methods will give you far more rigorous tools for making exactly this kind of comparison at scale.