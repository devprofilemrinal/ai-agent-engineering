# Phase 04 — Memory Systems

## Why this phase exists

Phase 03 taught you retrieval: fetching relevant *external* material (documents, policies, code) into a stateless model's context. Memory is the same underlying problem — selective, deliberate re-injection of information into a stateless call — applied to a different source: the conversation's own past, rather than an external corpus. If Phase 03 didn't exist first, memory would look like a completely separate topic requiring its own new mental model. Because it does exist first, memory is really "retrieval, but the corpus is your own conversation history" — recognizing that connection is the entire point of placing this phase here, immediately after retrieval rather than, say, immediately after Phase 02.

Phase 02's `ConversationState` (file 2) already gave you the simplest possible memory strategy: keep everything, resend everything, every turn. That works, and Phase 01's cost mechanics already told you exactly why it doesn't scale — linear cost growth with conversation length, and a real risk of burying important early facts under later, less relevant content (Phase 01, file 3). This phase is about the actual engineering decision of what to keep, what to compress, and what to only fetch when it's specifically needed — the same three-way choice a backend engineer already makes constantly when deciding what belongs in a fast in-memory cache, what belongs in a compressed summary table, and what's fine to leave in cold storage and query only on demand.

## What this phase covers

1. `01-short-term-vs-long-term-memory.md` — the fundamental distinction, and why conflating them causes real production problems.
2. `02-conversation-buffer-and-summarization-memory.md` — the two most common short-term strategies, their exact failure modes, and how to implement both in Java.
3. `03-vector-store-as-long-term-memory.md` — reusing Phase 03's exact retrieval machinery, now pointed at past conversation turns and user facts instead of documents.
4. `04-memory-in-langchain4j-and-spring-ai.md` — how both frameworks package these strategies, compared directly.
5. `05-project-persistent-memory-assistant.md` — a working assistant that remembers user-specific facts across entirely separate sessions, not just within one conversation.

## Prerequisites

Phase 02 (statelessness, and the `ConversationState` baseline this phase improves on) and Phase 03 (vector similarity and retrieval, which file 3 of this phase reuses directly rather than reintroducing).

## What you gain from this phase

The ability to look at a given conversational pattern — a quick one-off question, a long troubleshooting session, a personal assistant used across weeks — and choose the right memory strategy deliberately, rather than defaulting to "keep everything" until cost or quality problems force a reconsideration. This matters immediately in Phase 05 and beyond: every agent you build from here on has to manage its own growing history of tool calls and results, and unmanaged agent memory is a more urgent version of the same problem this phase solves for ordinary conversation.