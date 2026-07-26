# Phase 08 — Agent Frameworks in Java

## Why this phase exists, and why only now

Every phase from 02 through 07 was deliberately framework-free. That was not an arbitrary constraint or a purist exercise — it was a specific bet about how learning transfers. A framework's agent abstraction, encountered first, teaches you its API surface: which builder method to call, which annotation to add. It does not teach you what happens when that abstraction misbehaves, because the abstraction itself is the only thing you've ever seen. Having now hand-built the wire protocol (Phase 02), retrieval (Phase 03), memory (Phase 04), tool validation and safety (Phase 05), structured-output reliability (Phase 06), and a complete, budgeted, multi-pattern agent loop (Phase 07), you are in a position to look at LangChain4j's `AiServices` or Spring AI's `ChatClient` and recognize, concretely, exactly which of those seven phases' worth of mechanics each line of framework code is executing on your behalf. That recognition is the entire point of this phase, and it's only possible because of the order this handbook has followed.

This matters for a very practical reason, not just intellectual completeness. Frameworks fail. An `AiServices`-built agent will, at some point in a real system, do something surprising — loop longer than expected, fail to retain a fact it should have remembered, execute a tool call your validation logic should have blocked. When that happens, you have two possible postures: treat the framework as an opaque box and file a GitHub issue, or open up what you now know it must be doing internally — a ReAct-shaped loop (Phase 07), tool dispatch with a schema derived from your method signature (Phase 05), a memory strategy with some default window size (Phase 04) — and reason your way to the actual cause. This phase is designed to make the second posture available to you, on day one of adopting either framework, rather than something you'd only develop after months of production incidents.

## What this phase covers

1. **`01-langchain4j-ai-services-abstraction.md`** — how `AiServices` composes an interface-driven agent from tools, memory, and a chat model, mapped explicitly back to the loop you hand-built in Phase 07, file 4.
2. **`02-spring-ai-chatclient-and-advisors.md`** — Spring AI's `ChatClient` fluent API and its advisor chain, a genuinely distinctive architectural idea worth understanding on its own terms, again mapped back to Phase 07's hand-built mechanics.
3. **`03-choosing-langchain4j-vs-spring-ai.md`** — a direct, opinionated comparison: not a neutral feature table, but real guidance on which fits which team and codebase, including each framework's honest rough edges as of today.
4. **`04-project-framework-migration-exercise.md`** — taking the hand-built research agent from Phase 07's project and rebuilding it twice, once in each framework, so the comparison in file 3 is something you've verified yourself rather than taken on this handbook's word.

## Prerequisites

All of Phase 07 (the complete hand-built agent loop this phase's frameworks are compared against), Phase 05 (tool schemas and validation, which both frameworks' tool-calling annotations wrap), and Phase 04 (memory strategies, which both frameworks' `ChatMemory` abstractions wrap).

## What you gain from this phase

The ability to pick up either framework and immediately know what it's doing beneath its API surface — which means you can predict its behavior in new situations rather than only in the ones a tutorial happened to cover, debug it confidently when it misbehaves in production, and make a real, defensible choice between the two for a given team and codebase rather than picking one based on which had more search results or a more polished getting-started guide.