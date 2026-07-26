# Phase 07 Project — Hand-Rolled Research Agent

## Scenario

Your team occasionally needs a quick, synthesized answer to a question that requires pulling together information from several sources and reasoning across them — not a single lookup, but a genuine multi-step investigation. This project builds exactly that: a research agent, with zero framework dependency, that plans its approach, searches for information across multiple angles, reasons over what it finds, and produces a synthesized final answer with cited sources. This is the most substantial project in the handbook so far, deliberately — it's the first project where every earlier phase's mechanics genuinely have to work together under real operating constraints, not as isolated demonstrations.

## Functional requirements

1. **Build the complete, budgeted agent loop from file 4** — iteration cap, cost cap, wall-clock time cap, and the memory-management strategy for the loop's own growing history, not a simplified version without these safeguards.
2. **Implement at least two distinct research tools**: a search tool (against a real search API, or a fixed, realistic simulated corpus if you don't have one readily available) and a fetch/read tool that retrieves the full content of a specific result the search tool returned — mirroring the real two-step pattern of "find candidates, then read the promising ones" rather than a single tool that does both at once.
3. **Choose and justify a reasoning pattern** for this task — ReAct (file 1), plan-and-execute (file 2), or a hybrid — based on the actual shape of a research task: is the sequence of needed searches genuinely knowable upfront, or does it depend on what earlier searches reveal? Your choice must be defensible against the decision framework laid out in file 2.
4. **Add a reflection pass (file 3) on the final synthesized answer only**, not on every intermediate step — critiquing the final answer against explicit criteria: does it directly answer the original question, is every claim traceable to a specific source the agent actually retrieved, and does it avoid stating anything as fact that wasn't found in a retrieved source.
5. **Produce a final answer with source attribution** — every substantive claim in the synthesized answer should be traceable back to which specific search result or fetched page it came from, not presented as free-floating, unsourced assertion.
6. **Handle a genuinely open-ended, multi-hop research question** as your primary test case — something that can't be answered by a single search result alone and requires synthesizing across at least three distinct sources, proving the agent's iterative or planned multi-step structure is actually doing real work, not just wrapping a single search call in unnecessary ceremony.

## Constraints

- No framework — this project uses the plain Java agent loop from file 4 directly, built on the Phase 02 client, Phase 05 tool infrastructure, and Phase 06 validation pipeline, exactly as this phase has built them throughout.
- The agent must halt cleanly and report a partial result when any budget (iteration, cost, or time) is exceeded, rather than crashing or hanging — demonstrate this explicitly by deliberately setting an unreasonably tight budget for one test run and confirming the halt-and-report behavior functions correctly.
- The reflection pass's critique criteria must be explicit and checkable (per file 3's warning against vague, open-ended critique prompts) — "is this a good answer?" does not satisfy this requirement; "does every claim cite a specific retrieved source, and does the answer directly address the original question?" does.

## What "done" looks like

- A full execution trace for the primary multi-hop research question, showing each reasoning step, each tool call and its result, and the final reflection pass's critique (even if the critique found no issues) — demonstrating the complete loop from file 4 operating end to end, not just a final answer with no visibility into how it was produced.
- A final answer where every substantive claim can be matched to a specific line in the execution trace showing which source it came from.
- A demonstrated budget-halt: a deliberately constrained run (e.g., `maxIterations = 2` on a question that genuinely needs more steps) that halts cleanly with a clear, informative partial result rather than failing silently or looping past its stated limit.
- A short written justification for your reasoning-pattern choice (ReAct, plan-and-execute, or hybrid), referencing the specific characteristics of a research task that led you to that choice rather than an arbitrary preference.

## Extension

Add a second reflection dimension: after the agent produces its plan (if using plan-and-execute) or after its first two or three tool calls (if using ReAct), have it critique its *own research strategy so far* — "am I actually converging on an answer to the original question, or have I drifted into a tangential sub-topic?" — and allow it to explicitly redirect if the critique finds it has. This is a direct preview of the kind of self-monitoring that becomes essential once Phase 09 introduces multiple agents whose combined behavior needs to stay coherent with an original objective across many more steps than a single agent's loop typically runs.