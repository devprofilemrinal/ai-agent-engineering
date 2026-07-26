# Phase 03 — Context Augmentation and Retrieval

## Why this phase exists

Phase 01 established a hard fact: a model only knows what's inside its context window, and hallucination (file 5 of that phase) is what happens when the correct fact isn't there and the model produces a plausible guess instead. Phase 02 gave you a working client that can send whatever you put into that context window, but did nothing to solve *what* should go into it. This phase closes that gap.

"RAG" (Retrieval-Augmented Generation) is the industry name for the pattern this phase teaches, but the name itself isn't the point — the point is a general engineering principle: **when a stateless, context-limited function needs facts it doesn't reliably have memorized, you fetch those facts and put them directly into its input.** This is precisely the same principle behind, say, a stateless Lambda function that needs data — it doesn't "remember" anything between invocations either, so you pass it what it needs or have it look it up. RAG is that same idea, specialized to the case where "what it needs" is unstructured text and "looking it up" means finding the most semantically relevant chunks rather than an exact key match.

This phase deliberately teaches the underlying mechanics — vector similarity, chunking, retrieval evaluation — by hand, in plain Java, before showing how LangChain4j and Spring AI package the same mechanics. That ordering is intentional and consistent with the rest of this handbook: by the time you read the framework-specific files, you'll recognize exactly what they're doing internally, rather than trusting an `EmbeddingStore.findRelevant(...)` call as unexplained magic.

## What this phase covers

1. `01-context-engineering-as-a-discipline.md` — reframing "prompt engineering" and "RAG" as instances of one general practice: deciding what goes into a limited, valuable context window.
2. `02-vector-similarity-from-first-principles.md` — the actual math (cosine similarity) behind "these two pieces of text are related," computed by hand in Java before any library does it for you.
3. `03-chunking-strategies.md` — how you split documents before embedding them, and why this single decision often has more impact on retrieval quality than which embedding model you pick.
4. `04-embedding-models-and-stores.md` — practical embedding model selection, and what a vector store actually is (and isn't).
5. `05-retrieval-quality-and-evaluation.md` — how to know whether your retrieval step is actually working, rather than assuming it is because the demo looked fine.
6. `06-langchain4j-rag-building-blocks.md` — how LangChain4j packages the concepts above into `DocumentSplitter`, `EmbeddingStore`, and related abstractions.
7. `07-spring-ai-vector-store-abstraction.md` — the equivalent Spring AI abstractions, and how they compare.
8. `08-project-java-rag-knowledge-assistant.md` — a complete Java RAG pipeline, built first by hand and then reimplemented with a framework, over a real document set.

## Prerequisites

Phase 01 (embeddings as a concept, tokens, context window limits) and Phase 02 (a working, resilient client to actually send augmented prompts through).

## What you gain from this phase

The ability to build a retrieval pipeline from raw vector math, evaluate whether it's actually retrieving the right material, and recognize exactly what a framework's RAG abstraction is doing for you underneath. This is also your first real exposure to a system with a component *outside* the LLM itself (an embedding model, a vector store) that you're responsible for engineering correctly — a preview of the multi-component thinking Phase 09 (multi-agent orchestration) will require at a larger scale.