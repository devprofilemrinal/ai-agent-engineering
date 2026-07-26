# Memory in LangChain4j and Spring AI

## Why this exists

You've now hand-built every memory strategy this phase covers: bounded buffers, summarization, and vector-backed long-term recall. This file shows how both frameworks package these exact strategies — and, as with Phase 03's equivalent file, the goal in reading it is recognition, not first-time learning: you should be able to point at each framework abstraction and say precisely which hand-built mechanism it corresponds to.

## LangChain4j's memory abstractions

LangChain4j exposes short-term memory through a `ChatMemory` interface, with `MessageWindowChatMemory` as the direct equivalent of this phase's `BufferMemory`:

```java
import dev.langchain4j.memory.ChatMemory;
import dev.langchain4j.memory.chat.MessageWindowChatMemory;
import dev.langchain4j.data.message.UserMessage;
import dev.langchain4j.data.message.AiMessage;

ChatMemory chatMemory = MessageWindowChatMemory.withMaxMessages(20); // same job as BufferMemory(20)

chatMemory.add(UserMessage.from("What does idempotency mean?"));
chatMemory.add(AiMessage.from("It means..."));

List<ChatMessage> currentHistory = chatMemory.messages(); // same job as currentHistory()
```

For summarization-style memory, LangChain4j does not ship a single drop-in "summarizing memory" class equivalent to the `SummarizationMemory` you hand-built — this is a case where the framework leaves you closer to the hand-built approach than Phase 03's RAG abstractions did, and you'd typically implement the summarization-triggering logic yourself around `ChatMemory`, calling the chat model directly for compression exactly as file 2 of this phase demonstrated. Worth noting explicitly: not every framework abstracts every pattern in this handbook to the same degree, and recognizing where a framework leaves you to hand-roll something is as important as recognizing where it doesn't.

For long-term, vector-backed memory, LangChain4j doesn't provide a separate "long-term memory" abstraction distinct from what you already saw in Phase 03 — it's the same `EmbeddingStore`/`EmbeddingModel`/`EmbeddingStoreContentRetriever` combination from Phase 03, file 6, simply pointed at stored facts instead of documents, exactly as this phase's file 3 built it by hand. This is a deliberate design observation worth sitting with: LangChain4j doesn't treat "retrieval" and "long-term memory" as separate concepts at the API level, because — as this phase has argued throughout — they aren't separate concepts; they're the same mechanism applied to different content sources.

## Spring AI's memory abstractions

Spring AI centers on a `ChatMemory` interface as well, conceptually similar in purpose, with implementations following Spring's convention-driven style:

```java
import org.springframework.ai.chat.memory.ChatMemory;
import org.springframework.ai.chat.memory.MessageWindowChatMemory;

ChatMemory chatMemory = MessageWindowChatMemory.builder()
    .maxMessages(20)
    .build();

chatMemory.add(conversationId, new UserMessage("What does idempotency mean?"));
List<Message> history = chatMemory.get(conversationId);
```

Notice the explicit `conversationId` parameter threaded through both calls — Spring AI's `ChatMemory` is designed from the outset to manage *multiple distinct conversations* keyed by an identifier, whereas the LangChain4j example above manages one `ChatMemory` instance per conversation directly. Neither design is wrong; they reflect the same convention-versus-explicit-composition difference Phase 03, file 7 already identified — Spring AI baking in multi-conversation key management by default, LangChain4j leaving you to instantiate a separate memory object per conversation yourself.

Spring AI integrates memory into the request pipeline via its **advisor** mechanism — a chain of interceptors that can inspect and modify a request before it's sent and a response after it's received, which will be covered fully in Phase 08 as one of Spring AI's more distinctive architectural ideas:

```java
ChatClient chatClient = ChatClient.builder(chatModel)
    .defaultAdvisors(MessageChatMemoryAdvisor.builder(chatMemory).build())
    .build();

// Memory is now automatically read before, and updated after, every call —
// the manual add()/currentHistory() plumbing from file 2 is handled by the advisor.
```

This `MessageChatMemoryAdvisor` is doing, automatically, exactly what your hand-built `BufferMemory.add()` and `currentHistory()` calls did explicitly in file 2 — the difference is *where* that logic lives: explicit in your own code versus implicit inside a configured advisor. Being able to see that equivalence is precisely why this phase insisted on hand-building the mechanism first.

## What neither framework removes your responsibility for

- **Deciding which memory strategy fits your use case** (file 1's short-term/long-term distinction, and the buffer-vs-summarization trade-off from file 2) is a design decision no framework makes for you — both frameworks give you the buffer-window mechanism readily, and leave summarization and long-term extraction largely to your own implementation.
- **The extraction step for long-term memory** (deciding *what* is durable and worth storing, from file 3) is not something either framework provides out of the box — it's a genuine engineering decision specific to your application, and you'll write that extraction logic yourself in either ecosystem.
- **Multi-tenant correctness** (the `userId`/metadata filtering discussed in file 3) is still entirely your responsibility in both frameworks — a shared `ChatMemory` or vector store, misconfigured, will happily leak one user's context into another's conversation regardless of which framework you're using.

## Trade-offs and when this matters most

- If your application already needs to manage many concurrent, distinctly-keyed conversations (a typical multi-user Spring Boot service), Spring AI's `conversationId`-first design maps onto that requirement with less custom wiring than LangChain4j's per-instance memory objects.
- If your team wants closer, more explicit control over exactly when and how summarization or extraction happens, hand-rolling around either framework's basic `ChatMemory` (as this phase did throughout) remains the right approach in either ecosystem — neither framework's abstraction should be treated as a reason to stop understanding the underlying strategy.
- Don't assume a framework's default `ChatMemory` window size or strategy is appropriate for your conversation patterns without deliberately choosing it, the same way you wouldn't accept a database connection pool's default size without considering your actual load.

## Why this matters next

You've now completed this phase's full arc: the short-term/long-term distinction, both short-term strategies hand-built, long-term memory reusing Phase 03's retrieval machinery, and both frameworks' packaging of all of it. The final project asks you to build a real, persistent assistant that demonstrably remembers a user across entirely separate sessions — not just within one conversation, proving out this phase's central distinction in a working artifact rather than leaving it as theory.