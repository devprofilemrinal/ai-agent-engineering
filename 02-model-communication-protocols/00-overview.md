# Phase 02 — Model Communication Protocols

## Why this phase exists

Phase 01 gave you the internal mechanics: tokens, embeddings, attention, sampling, hallucination, cost. All of that explains what happens *inside* the model. This phase is about everything that happens *around* it — the actual HTTP calls, JSON shapes, authentication, streaming, and failure handling you'll write real Java code for. This is the phase where "AI Engineering" and "backend engineering" stop being separate disciplines and become the same discipline applied to a new kind of upstream dependency.

The framing to hold onto throughout this entire phase: an LLM API is a REST API with a few genuinely new behaviors layered on top of otherwise ordinary HTTP semantics. Your job in this phase is to learn precisely where the "ordinary" ends and the "genuinely new" begins, and to write client code — by hand, with no framework — that handles both correctly. LangChain4j and Spring AI (Phase 08) will eventually wrap all of this for you, but you write it by hand first so that when a framework's abstraction leaks or misbehaves in production, you know exactly what's underneath it and can debug past it.

## What this phase covers, and why in this order

1. **`01-rest-anatomy-of-llm-apis.md`** — the shape of the request and response bodies, before anything else, because every other file in this phase refers back to this shape.
2. **`02-message-roles-and-statelessness.md`** — how conversation history is represented and why the server remembers nothing between calls, which is the direct cause of the cost-compounding behavior from Phase 01.
3. **`03-authentication-and-secrets.md`** — how credentials are attached to requests, and the operational discipline around them, before you write any code that could leak one.
4. **`04-streaming-with-sse.md`** — how a response arrives incrementally, tying directly back to autoregressive generation from Phase 01.
5. **`05-function-calling-wire-protocol.md`** — the exact JSON contract that lets a model ask your code to do something, which is the seed of every "agent" behavior in Phases 05 onward.
6. **`06-structured-output-json-mode.md`** — how to constrain a model's output to a schema at the protocol level, laying groundwork for Phase 06's reliability patterns.
7. **`07-provider-differences-anthropic-vs-openai.md`** — where the "standard" you've just learned actually diverges between vendors, so you don't assume false portability.
8. **`08-java-http-client-implementation.md`** — writing all of the above as real, dependency-free Java using `java.net.http.HttpClient`.
9. **`09-resilience-retries-backoff-circuit-breakers.md`** — handling the specific failure modes of this API type (rate limits, overload, malformed output) with patterns you already know from distributed systems, adapted to new specifics.
10. **`10-project-java-cli-chat-client.md`** — a complete, working CLI chat client combining everything above, using nothing but the JDK.

## Prerequisites

Phase 01, specifically: tokens (to understand payload sizing), statelessness as introduced conceptually in Phase 00 (this phase makes it concrete at the JSON level), and autoregressive generation (to understand why streaming exists at all).

## What you gain from this phase

A working, resilient, dependency-free Java client capable of authenticating, sending conversation history, streaming a response, handling tool-call requests from the model, requesting structured output, and recovering from the specific failure modes this API type produces — built by hand, so that every later framework you adopt (Phase 08) is something you can reason about and debug rather than something you trust blindly.