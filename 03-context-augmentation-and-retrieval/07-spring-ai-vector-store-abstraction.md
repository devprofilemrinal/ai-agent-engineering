# Spring AI Vector Store Abstraction

## Why this exists

Having seen LangChain4j's take on packaging this phase's mechanics, this file covers Spring AI's equivalent — built on Spring's own conventions (auto-configuration, dependency injection, `@Bean`-defined components) rather than LangChain4j's more explicit, manually-composed builder chains. If your team's codebase is already a Spring Boot shop, this is very likely the more natural fit, and this file exists so you can judge that for yourself with a concrete side-by-side rather than taking that claim on faith.

## The core abstraction: `VectorStore`

Where LangChain4j exposes separate `EmbeddingStore` and `EmbeddingModel` interfaces you compose yourself, Spring AI centers on a single `VectorStore` interface that internally handles embedding as part of storing and searching — a more convention-driven, less explicit design, consistent with Spring's general philosophy of hiding wiring behind configuration:

```java
public interface VectorStore {
    void add(List<Document> documents);
    List<Document> similaritySearch(SearchRequest request);
    // additional methods for deletion, filtering, etc.
}
```

A working example, using an in-memory implementation for direct comparison with the LangChain4j version from the previous file:

```java
import org.springframework.ai.document.Document;
import org.springframework.ai.embedding.EmbeddingModel;
import org.springframework.ai.transformer.splitter.TokenTextSplitter;
import org.springframework.ai.vectorstore.SimpleVectorStore;
import org.springframework.ai.vectorstore.SearchRequest;

@Configuration
public class RagConfig {

    @Bean
    public VectorStore vectorStore(EmbeddingModel embeddingModel) {
        return SimpleVectorStore.builder(embeddingModel).build();
    }
}

@Service
public class DocumentIngestionService {

    private final VectorStore vectorStore;
    private final TokenTextSplitter splitter = new TokenTextSplitter();

    public DocumentIngestionService(VectorStore vectorStore) {
        this.vectorStore = vectorStore;
    }

    public void ingest(String rawText, Map<String, Object> metadata) {
        Document document = new Document(rawText, metadata);
        List<Document> chunks = splitter.apply(List.of(document)); // your file 3 chunking, framework-provided
        vectorStore.add(chunks); // embeds and stores, in one call
    }
}

@Service
public class RetrievalService {

    private final VectorStore vectorStore;

    public RetrievalService(VectorStore vectorStore) {
        this.vectorStore = vectorStore;
    }

    public List<Document> retrieve(String query, int k) {
        SearchRequest request = SearchRequest.builder()
            .query(query)
            .topK(k)
            .similarityThreshold(0.6) // same purpose as LangChain4j's minScore
            .build();
        return vectorStore.similaritySearch(request);
    }
}
```

Notice `vectorStore.add(chunks)` performs embedding *and* storage in one call — Spring AI's `VectorStore` implementation holds a reference to the `EmbeddingModel` internally (injected as a constructor argument to `SimpleVectorStore.builder(...)`), so your service code never touches a raw embedding vector directly. This is a meaningfully different level of abstraction than LangChain4j's approach in the previous file, where you explicitly held and passed an `EmbeddingModel` instance to both the ingestor and the retriever separately.

## The philosophical difference this reflects

LangChain4j's design keeps each stage (splitting, embedding, storing, retrieving) as separate, explicitly composed objects you wire together yourself — closer to the mental model of manually assembling a pipeline. Spring AI leans further into hiding that composition behind fewer, higher-level interfaces and Spring's dependency injection — closer to the mental model of declaring "I want a `VectorStore`" and letting the framework's auto-configuration assemble the pieces behind it, the same convention-over-configuration instinct you already know from Spring Boot's general approach to, say, `DataSource` configuration.

Neither approach is objectively better — they represent different points on the explicit-composition-versus-convention spectrum, and which one fits better depends on your team and codebase, covered as a direct decision point in Phase 08.

## Switching the underlying store: the actual abstraction payoff

Exactly like LangChain4j's `EmbeddingStore` interface, Spring AI's `VectorStore` interface is implemented by numerous backends (PGVector, Redis, Elasticsearch, and others), and swapping between them is a matter of changing which `VectorStore` bean you configure, not rewriting `DocumentIngestionService` or `RetrievalService` at all — the abstraction seam from Phase 02, file 7's discussion applies just as directly here:

```java
@Bean
public VectorStore vectorStore(EmbeddingModel embeddingModel, JdbcTemplate jdbcTemplate) {
    return PgVectorStore.builder(jdbcTemplate, embeddingModel).build();
    // same interface, now backed by a real persistent, indexed vector database
}
```

## What Spring AI still leaves entirely to you

- **Chunk size and splitting strategy tuning (file 3)** — `TokenTextSplitter`'s defaults are a starting point, not a guarantee of good retrieval for your specific corpus.
- **Evaluation (file 5)** — `similarityThreshold` is a mechanism Spring AI exposes, same as LangChain4j's `minScore`, but choosing the right value and building a golden set to validate it is unchanged, framework-independent work.
- **Cost and token awareness (Phase 01)** — every `vectorStore.add(...)` call is still real embedding API calls with real token cost; the convenience of one method call hides the mechanics, not the economics.

## Trade-offs and when this matters most

- If your team already has Spring Boot conventions, dependency injection patterns, and configuration management deeply embedded in its engineering culture, Spring AI's `VectorStore` will likely integrate with less friction and less new convention to learn than LangChain4j's more explicit composition style.
- If you value seeing and controlling each pipeline stage explicitly — useful when debugging exactly where a retrieval pipeline is misbehaving, or when you need fine control that a higher-level convenience method doesn't expose — LangChain4j's more granular interfaces can be an advantage, at the cost of writing and wiring more of the composition yourself.
- Don't choose one over the other purely on the basis of which is "more popular" in general AI-engineering discourse — that discourse is dominated by the wider (Python) LangChain ecosystem, not necessarily by which is the better fit for your specific Java/Spring codebase; Phase 08 covers this decision properly, with both frameworks used to build the same complete agent.

## Why this matters next

You've now seen both frameworks' RAG abstractions, side by side with the hand-built version from earlier in this phase. The final project asks you to actually build and compare these approaches yourself, on a real document set, closing out this phase's entire progression from raw vector math to a working, evaluated retrieval pipeline.