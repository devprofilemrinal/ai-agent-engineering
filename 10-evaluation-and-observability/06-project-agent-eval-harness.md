# Phase 10 Project — Agent Evaluation Harness

## Scenario

Your team's multi-agent research pipeline from Phase 09 works — you've seen it produce good answers in testing. But "works, as far as I've seen" is exactly the weak evidence Phase 00 warned you never to over-trust for a probabilistic system, and it's not something you could respond to a skeptical stakeholder with confidently, let alone something you'd want to be your only signal before a production rollout. This project builds the complete evaluation harness this phase has been assembling piece by piece — tracing, safe logging, calibrated judging, repeated-run regression testing, and cost/latency aggregation — and applies all of it to the Phase 09 pipeline, so you have actual evidence, not impression, about how well it performs and how it degrades under real variance.

## Functional requirements

1. **Instrument the complete Phase 09 multi-agent pipeline with tracing** (file 1) — every reasoning call, tool execution, and inter-agent handoff as a properly nested span, including the supervisor/graph-level spans and each worker agent's own nested trace, exactly per file 1's cross-agent tracing diagram.
2. **Apply safe logging discipline** (file 2) to everything captured — build the purpose-built, intentionally-lossy logging representation appropriate to this pipeline's actual content (search results and summaries, in this case), and implement the selective full-fidelity capture pattern for at least budget-halted and validation-failed runs.
3. **Build a calibrated LLM-as-judge** (file 3) for the pipeline's final synthesized answers, scored against explicit criteria (does the answer address the original question, is every claim traceable to a retrieved source, is the answer appropriately scoped to what was actually found). Calibrate it against at least 10 human-scored examples before trusting its output, and report the correlation between your judge's scores and your own human scoring.
4. **Build a golden set of at least 12 realistic research tasks**, each with defined deterministic checks (did the agent actually call its tools rather than guessing, did the graph correctly route through the insufficient-material branch when it should have) and judge-based criteria, per file 4's `AgentTestCase` structure.
5. **Run the full golden set with repetition** (file 4) — at least 5 repetitions per test case — and report pass rates per case, not single pass/fail results, honestly surfacing any test cases with a meaningfully lower pass rate than the others.
6. **Produce an aggregated cost and latency report** (file 5) across the full evaluation run: total cost, cost broken down by agent type (search vs. summarizer vs. supervisor overhead) and by tool, p50/p95/p99 latency, and the budget-halt rate observed across all repetitions.

## Constraints

- The judge must be calibrated against real human scoring before its results are used in the regression suite — an uncalibrated judge's scores, per file 3's leniency-bias warning, cannot be trusted as a meaningful pass/fail signal on their own.
- At least one golden-set test case must include a deterministic behavioral check (not just a content check) — verifying, for example, that the agent actually invoked its search tools rather than producing a plausible-sounding answer without genuinely retrieving anything, per file 4's discussion of testing behavior, not just output.
- The full-fidelity capture mechanism (file 2) must be demonstrably gated — show explicitly that ordinary trace data does not contain full, unredacted prompt/response content, while the separate, flagged capture store does, for at least one deliberately flagged run.

## What "done" looks like

- A complete, navigable trace for at least one full pipeline run, showing every agent's reasoning steps, tool calls, and the supervisor/graph-level handoffs between them, retrievable and readable after the run has completed — not just visible while it was happening.
- A calibration report for your LLM judge: the human scores you assigned to your reference examples, the judge's scores on the same examples, and the resulting correlation or agreement rate, with an honest note about where the judge and human scoring diverged most, if anywhere.
- A per-test-case pass-rate table from the repeated golden-set run, with at least one case's pass rate below 100% honestly reported and investigated — explain, using your trace data, what caused the failures in the repetitions that didn't pass, rather than only reporting the aggregate number.
- A cost and latency report broken down by agent type and by tool, identifying which specific component of the multi-agent pipeline is the largest cost driver, with your own brief assessment of whether that cost is justified by the value that component provides (tying back to Phase 09, file 1's supervisor-worker justification question, now backed by actual measured data instead of the earlier, pre-measurement judgment call).

## Extension

Take one golden-set test case that showed a pass rate below 100% in your repeated runs, use the trace data to diagnose the specific cause of the failing repetitions, make one targeted change (a prompt adjustment, a tool description improvement per Phase 05 file 1, a budget recalibration per Phase 07 file 4), and re-run the repeated evaluation for that specific case to measure whether the pass rate genuinely improved. This closes the full loop this phase has been building toward: not just detecting a problem, but using the same harness to verify a fix actually worked — the exact discipline that separates a team that can maintain a production agent system from one that can only build one.