# Phase 08 Project — Framework Migration Exercise

## Scenario

Your team has been running the hand-built research agent from Phase 07 in production for a while, and it works, but the hand-rolled loop is becoming a maintenance burden as more engineers need to extend it — everyone touching the codebase has to first understand several hundred lines of custom loop, budget, and dispatch logic before they can safely add a feature. Leadership wants to know: would adopting a framework actually reduce this burden, and which one? This project answers that question empirically rather than by opinion — you rebuild the exact same agent twice, once in each framework, and produce a real, evidence-based comparison rather than a guess.

## Functional requirements

1. **Rebuild Phase 07's research agent using LangChain4j's `AiServices`** (file 1), preserving the same tool set (search and fetch), the same task-handling capability (multi-hop research questions), and, critically, reintroducing Phase 07, file 4's budget safeguards (iteration, cost, and wall-clock limits) explicitly, since `AiServices` does not provide them by default.
2. **Rebuild the same agent again using Spring AI's `ChatClient` and advisor chain** (file 2), with the same tool set and task-handling capability, this time implementing the budget safeguards as a custom advisor rather than a wrapper function, per file 2's `BudgetEnforcingAdvisor` pattern.
3. **Run all three implementations** — the original Phase 07 hand-built version, the LangChain4j version, and the Spring AI version — against the identical set of research questions used in Phase 07's project, including at least one question deliberately designed to test budget-limit enforcement (a question complex enough, or a budget deliberately tight enough, to trigger a halt).
4. **Measure and compare, concretely, not impressionistically**: total lines of code for each implementation (excluding the identical tool implementations, which don't change across versions), whether each version's budget enforcement actually triggers correctly on the stress-test question, and any observable difference in final answer quality or behavior across the three.
5. **Produce a written comparison** addressing, specifically: which implementation was fastest to build from the working hand-built version, which made the budget-reintroduction requirement easiest to implement cleanly, and which you would recommend for this specific team's codebase — referencing file 3's decision framework explicitly rather than a general impression.

## Constraints

- The tool implementations themselves (the actual search and fetch logic) must be identical, or as close to identical as each framework's tool-registration mechanism allows, across all three versions — the comparison is about the framework's orchestration and safety-reintroduction burden, not about different underlying tool quality.
- The budget-enforcement requirement is not optional in either framework version — a version without working iteration/cost/time limits does not satisfy this project's requirements, since the entire point is comparing how cleanly each framework lets you reintroduce Phase 07's safeguards, not whether you can skip them.
- All three versions must be run against the identical research questions, including the deliberately budget-stressing one, so the comparison in the final report is genuinely apples-to-apples.

## What "done" looks like

- Three working implementations of the same research agent, each demonstrably completing the same non-trivial multi-hop research questions with comparable final-answer quality.
- A demonstrated budget halt in both the LangChain4j and Spring AI versions on the same stress-test question that Phase 07's original hand-built version was shown to halt on, proving the safeguard was genuinely reintroduced in both, not merely claimed.
- A written comparison report with concrete, specific observations — not "Spring AI felt more Spring-like" but "reintroducing the iteration budget in Spring AI took N lines as a reusable advisor, versus M lines as a wrapper function in LangChain4j, and the advisor version was independently unit-testable without invoking a real chat model, while the wrapper version was not" (or your own genuine, specific findings — the point is specificity, not agreeing with this handbook's own conclusions).
- A final, explicit recommendation for which framework this specific hypothetical team should adopt, referencing the decision framework from file 3.

## Extension

Take whichever framework your comparison favored and use it to add one genuinely new capability to the research agent that wasn't in Phase 07's original scope — for example, a second reflection pass (Phase 07, file 3) that runs specifically on the *plan* before execution begins, not just on the final answer. Notice, concretely, whether the framework's abstraction (declarative `AiServices` interface, or an explicit Spring AI advisor) made this extension easier or harder to add cleanly than it would have been in the original hand-built loop — this is the real, ongoing test of whether a framework's abstraction earns its convenience over time, not just at initial build.