# Phase 05 Project — Java DevOps Tool Agent

## Scenario

Your team wants a chat-driven assistant on-call engineers can use during an incident to quickly check service health and, when needed, take a small set of well-defined remediation actions — without giving it unrestricted access to run arbitrary commands against production. This project builds exactly that, against a real or realistically simulated Kubernetes-style API, and every safety discipline from this phase exists specifically because this is the first project in the handbook where a poorly-built agent could plausibly cause real (or realistically simulated) damage.

## Functional requirements

1. **Implement at least three narrow, single-purpose tools** (per file 1's guidance against broad catch-all tools): one read-only (e.g., `get_pod_status`), one moderately disruptive but recoverable (e.g., `restart_pod`), and one you deliberately treat as high-consequence (e.g., `scale_deployment` to zero replicas, or an equivalent action in whatever API you're working against).
2. **Write full JSON Schema definitions with constrained parameters** where possible (e.g., an `enum` for namespace, per file 1), not free-form strings for anything that has a fixed, known set of valid values.
3. **Implement all three validation layers from file 3**: schema validation, business-rule validation (e.g., the requesting user is only authorized for specific namespaces), and a sandboxing decision — document explicitly what infrastructure-level restriction (real or simulated) backs each tool, not just the application-level check.
4. **Implement confirmation gating (file 4) for the disruptive and high-consequence tools**, with a prompt that names the specific action and target rather than a generic approval request, and no gate at all for the read-only tool — demonstrating the selective-gating principle rather than gating everything uniformly.
5. **Handle a validation failure gracefully**, returning a proper `tool_result` error back to the model (per file 3) rather than crashing, and confirm the model can recover — for example, asking the user for a corrected namespace after being told the one it tried doesn't exist.
6. **Log every tool call attempt** — tool name, arguments, validation outcome, confirmation outcome, and final execution result — as a simple audit trail, since this is exactly the kind of record a real on-call tool would need for post-incident review.

## Constraints

- Do not implement a single broad "run any command" tool, even as a shortcut — the point of this project is narrow, validated, individually-gated tools, per file 1's explicit guidance against that design.
- The high-consequence tool must genuinely require explicit confirmation in every test run — no code path that allows it to execute without an approval step, even during testing.
- You may build this by hand (Phase 02's client plus this phase's validation/confirmation logic) or using LangChain4j or Spring AI's `@Tool` annotations (file 5) — but if using a framework, the validation, sandboxing, and confirmation logic must still be explicitly present inside your tool methods, not assumed to be handled by the framework.

## What "done" looks like

- A session where the read-only tool executes immediately with no confirmation prompt, the moderately disruptive tool prompts for and requires explicit approval, and the high-consequence tool's prompt clearly states the specific, named action about to be taken.
- A deliberately malformed or unauthorized request (e.g., asking to act on a namespace the simulated user isn't authorized for) is caught by business-rule validation and reported back to the model as a proper `tool_result` error, with the model producing a sensible follow-up response rather than the interaction crashing.
- A complete audit log entry for every tool call attempt made during a test session, including at least one rejected attempt and one approved one.

## Extension

Add a second, distinct simulated user role with more limited namespace authorization than the first, and confirm the same tool calls are correctly allowed or denied depending on which user is driving the session — this is a direct rehearsal for the multi-tenant and role-based concerns that become central once Phase 09 introduces multiple agents potentially acting under different authorities within the same system.