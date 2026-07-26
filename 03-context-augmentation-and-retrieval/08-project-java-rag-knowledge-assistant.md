# Phase 03 Project — Java RAG Knowledge Assistant

## Scenario

Your team maintains a growing set of internal technical documentation — architecture decision records, runbooks, API references — and engineers keep asking the same questions in Slack that are already answered somewhere in those docs, just hard to find quickly. This project builds a small assistant that answers questions grounded in your own real documentation, combining the Phase 02 chat client with this phase's retrieval pipeline, and forces you to build it three ways so the comparison in files 6 and 7 stops being abstract.

## Functional requirements

1. **Ingest a real, non-trivial document set.** Use actual technical documentation you have access to — your own project's README/wiki, a public open-source project's docs, or similar — of at least a few thousand words across multiple documents, not a single short toy paragraph.
2. **Implement chunking, embedding, and storage by hand first**, using the plain Java approach from files 2, 3, and 4 — no framework — to produce a working baseline retrieval pipeline you fully understand end to end.
3. **Build a golden set** of at least 10 realistic questions about your document set, each with the specific chunk(s) you've manually confirmed answer it, per file 5.
4. **Measure precision and recall** of your hand-built pipeline against the golden set, and tune chunk size or `k` at least once based on what the measurement shows, demonstrating the tune-and-remeasure loop rather than tuning blindly.
5. **Wire retrieval into the Phase 02 chat client**: for each incoming question, retrieve the top-k relevant chunks, insert them into the request (a natural place is the `system` field or as prefixed context ahead of the user's question in `messages`), and send the augmented request through your existing resilient client.
6. **Rebuild the same pipeline using LangChain4j's abstractions** (file 6), and separately, **using Spring AI's abstractions** (file 7), reusing the same document set and golden set so the comparison is apples-to-apples.
7. **Produce a short written comparison**: for your specific corpus and question set, note any differences in retrieval quality, code volume, and ease of tuning between the three implementations.

## Constraints

- The hand-built version must not use any RAG-specific library — plain Java, your own cosine similarity implementation, your own chunking logic, and either the raw embeddings HTTP call or a locally-run embedding model.
- All three versions (hand-built, LangChain4j, Spring AI) must be evaluated against the *identical* golden set, so precision/recall numbers are actually comparable rather than measuring different things.
- The assistant must clearly indicate when no chunk meets a reasonable similarity threshold, rather than always answering confidently from whatever the top-k happens to return — tying back to file 4's `minScore`/`similarityThreshold` discussion and the retrieval-failure-mode discussion in file 6.

## What "done" looks like

- You can point to specific golden-set questions where retrieval initially failed, describe the chunking or `k` change you made in response, and show the precision/recall numbers improving as a result — not just a final working demo, but evidence of the tune-and-measure loop from file 5 actually happening.
- All three implementations answer the same golden-set questions with comparable groundedness, and your written comparison identifies at least one concrete difference between the three approaches beyond "they all work."
- Asking a question genuinely outside your document set's coverage produces a clear "I don't have information on that" response rather than a hallucinated answer — proving the similarity-threshold safeguard actually functions, not just that it's configured.

## Extension

Add source citations to the assistant's responses — surfacing which document and section each part of the answer came from, using the metadata discipline from file 4. This is also a natural point to informally preview Phase 04: notice that this assistant has no memory of earlier questions in the same session at all right now, and consider what would need to change for a follow-up question like "what about the staging environment?" to correctly resolve against a topic raised two questions earlier.