# LangChain4j RAG Building Blocks

## Why this exists

Every mechanic in this phase so far — chunking, embedding, storing, retrieving, evaluating — you now understand well enough to have implemented from scratch. This file shows how LangChain4j packages those exact mechanics into a small set of reusable interfaces. Read this file the way you'd read a library's source code after already understanding the algorithm it implements: not to learn the concept for the first time, but to see how a well-designed abstraction maps onto something you already know.

## The core abstractions, mapped directly onto what you've already built

LangChain4j organizes the RAG pipeline into a handful of composable interfaces, each corresponding to a stage you hand-built earlier in this phase:

| LangChain4j abstraction | What you already built by hand |
|---|---|
| `DocumentSplitter` | `paragraphChunk` / `fixedSizeChunk` (file 3) |
| `EmbeddingModel` | the raw `/v1/embeddings` HTTP call (file 4) |
| `EmbeddingStore<TextSegment>` | `InMemoryVectorStore` (file 4) |
| `EmbeddingStoreContentRetriever` | your `retrieveTopK` method plus the query-embedding step (file 2) |
| `RetrievalAugmentor` | the full pipeline: retrieve, then inject into the prompt sent to the chat model |

## A working example, side by side with what you built

```java
import dev.langchain4j.data.document.Document;
import dev.langchain4j.data.document.splitter.DocumentSplitters;
import dev.langchain4j.data.segment.TextSegment;
import dev.langchain4j.model.embedding.EmbeddingModel;
import dev.langchain4j.model.embedding.onnx.allminilm.AllMiniLmL6V2EmbeddingModel;
import dev.langchain4j.store.embedding.EmbeddingStore;
import dev.langchain4j.store.embedding.inmemory.InMemoryEmbeddingStore;
import dev.langchain4j.store.embedding.EmbeddingStoreIngestor;
import dev.langchain4j.rag.content.retriever.EmbeddingStoreContentRetriever;

// 1. Chunking — same job as your paragraphChunk method
var splitter = DocumentSplitters.recursive(500, 50); // chunk size, overlap — same parameters you tuned by hand

// 2. Embedding model — same job as your raw /v1/embeddings HTTP call
EmbeddingModel embeddingModel = new AllMiniLmL6V2EmbeddingModel();

// 3. Vector store — same job as your InMemoryVectorStore
EmbeddingStore<TextSegment> embeddingStore = new InMemoryEmbeddingStore<>();

// 4. Ingestion — chunk + embed + store, all three of your earlier stages composed together
EmbeddingStoreIngestor ingestor = EmbeddingStoreIngestor.builder()
    .documentSplitter(splitter)
    .embeddingModel(embeddingModel)
    .embeddingStore(embeddingStore)
    .build();

Document policyDoc = Document.from(policyDocumentText);
ingestor.ingest(policyDoc);

// 5. Retrieval — same job as your retrieveTopK method
var contentRetriever = EmbeddingStoreContentRetriever.builder()
    .embeddingStore(embeddingStore)
    .embeddingModel(embeddingModel)
    .maxResults(3) // your "k"
    .minScore(0.6) // a similarity floor, filtering out weak matches your hand-rolled version returned unconditionally
    .build();
```

Notice `minScore` — a detail your hand-rolled `retrieveTopK` didn't include. This is a genuinely useful addition, not just boilerplate: it filters out chunks below a similarity threshold entirely, rather than always returning exactly `k` chunks even when none of them are actually relevant (a real failure mode of your earlier naive implementation — if a query has no good match in the corpus at all, `retrieveTopK` still dutifully returns its top 3 *worst-available* matches, which can feed irrelevant material into generation just as harmfully as the buried-fact problem from Phase 01).

## What LangChain4j gives you that hand-rolling made you feel the absence of

- **Pluggable document loaders** for PDFs, web pages, and other formats, so `03-chunking-strategies.md`'s splitting logic can operate on real-world document formats without you writing a PDF-parsing layer yourself.
- **A wider library of embedding model integrations** than you'd want to hand-write HTTP clients for individually, including local, no-API-call embedding models (like the `AllMiniLmL6V2EmbeddingModel` above) — useful for development and testing without incurring API costs on every ingestion run.
- **A consistent interface across many vector store backends** — swapping `InMemoryEmbeddingStore` for a real, persistent vector database is a one-line change to which implementation you construct, not a rewrite of your retrieval logic, precisely the abstraction-seam discipline Phase 02's file 7 argued for at the LLM-client level, now applied at the retrieval level.

## What it does not remove your responsibility for

- **Evaluation (file 5) is still entirely on you.** LangChain4j gives you `minScore` as a mechanism, but choosing the right threshold, building your golden set, and measuring precision/recall against it is still your own engineering work — the framework provides levers, not judgment.
- **Chunk size and splitting strategy still need to be tuned for your specific corpus**, per file 3 — `DocumentSplitters.recursive(500, 50)` is a starting configuration, not a universally correct one.
- **Cost awareness (Phase 01) doesn't disappear** — every chunk ingested is still an embedding API call with real token cost, and every retrieval-augmented chat call still inserts real tokens into your context window, exactly as before; the framework doesn't change the underlying economics, only the amount of code you write to implement them.

## Trade-offs and when this matters most

- For quick prototypes and learning, LangChain4j's batteries-included document loaders and embedding model integrations meaningfully reduce boilerplate versus hand-writing everything.
- For production systems with specific evaluation, monitoring, or custom retrieval-ranking needs, you'll often still drop below the high-level `EmbeddingStoreIngestor`/`ContentRetriever` abstractions into the lower-level pieces (the embedding model and store interfaces directly) — which is only possible to do confidently because you understand, from this phase's earlier files, what those lower-level pieces are actually doing.
- Don't adopt LangChain4j's RAG abstractions as a substitute for understanding chunking and evaluation trade-offs — a framework will happily let you ingest badly-chunked documents and retrieve poorly without any warning; the responsibility for quality is still yours.

## Why this matters next

You've now seen how LangChain4j packages this phase's mechanics. The next file covers Spring AI's equivalent abstractions — built with different conventions, worth comparing directly rather than assuming they're interchangeable — before the final project asks you to build the same pipeline three ways: by hand, in LangChain4j, and (optionally, previewing Phase 08's fuller comparison) in Spring AI.