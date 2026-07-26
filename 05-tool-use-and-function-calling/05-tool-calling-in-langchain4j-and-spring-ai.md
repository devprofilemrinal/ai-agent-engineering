# Tool Calling in LangChain4j and Spring AI

## Why this exists

You've now hand-built every stage of the tool-calling discipline: schema design, the mechanism behind tool selection, validation, sandboxing, and confirmation gating. This file shows how both frameworks let you define and dispatch tools with far less boilerplate than hand-writing the JSON schema and dispatch logic yourself (Phase 02, file 5) — and, as with every framework file in this handbook, the goal is to see exactly what's being generated for you, and to check whether this file's safety patterns still have a natural place to live once the framework is doing more of the work.

## LangChain4j: annotation-driven tool definitions

LangChain4j lets you define a tool as a plain Java method, annotated with `@Tool`, and it generates the JSON Schema from the method's parameters automatically:

```java
import dev.langchain4j.agent.tool.Tool;
import dev.langchain4j.agent.tool.P;

public class KubernetesTools {

    @Tool("Retrieves the current status of a Kubernetes pod.")
    public String getPodStatus(
        @P("The Kubernetes namespace") String namespace,
        @P("The pod name") String podName
    ) {
        return kubernetesClient.getPodStatus(namespace, podName);
    }
}
```

This method's signature and its `@P` parameter descriptions are exactly what file 1's `input_schema` and parameter `description` fields were doing by hand — LangChain4j introspects the method at startup and constructs the schema for you. Wiring it into an agent is a matter of registering the tools object:

```java
Assistant assistant = AiServices.builder(Assistant.class)
    .chatModel(chatModel)
    .tools(new KubernetesTools())
    .build();

String response = assistant.chat("Is payments-service healthy in prod?");
// Behind the scenes: tool_use generated, method invoked directly, tool_result sent back, final answer returned
```

**What this hides that file 3 and file 4 required you to write explicitly:** the tool dispatch shown here calls `getPodStatus` directly, with no seam for schema re-validation, business-rule authorization, or a confirmation gate before execution. If you adopt this pattern as-is for anything beyond read-only tools, you need to add those layers back in yourself — typically by having the annotated method itself perform validation and, for consequential actions, call out to a confirmation mechanism before proceeding, rather than assuming the framework's convenience wiring provides safety guarantees it was never designed to provide.

```java
@Tool("Restarts a Kubernetes pod. Use only when explicitly requested and confirmed.")
public String restartPod(
    @P("The Kubernetes namespace") String namespace,
    @P("The pod name") String podName
) {
    authorizer.authorize(namespace, podName, currentUserId()); // file 3, still your responsibility
    if (!confirmationPrompter.confirm("Restart pod " + podName + " in " + namespace + "?")) { // file 4, still your responsibility
        return "Action cancelled by user.";
    }
    return kubernetesClient.restartPod(namespace, podName);
}
```

## Spring AI: `@Tool`-annotated methods with a similar shape

Spring AI's tool-calling mechanism is structurally very similar — annotate a method, register the object, and the framework generates the schema and handles dispatch:

```java
import org.springframework.ai.tool.annotation.Tool;
import org.springframework.ai.tool.annotation.ToolParam;

@Service
public class KubernetesTools {

    @Tool(description = "Retrieves the current status of a Kubernetes pod.")
    public String getPodStatus(
        @ToolParam(description = "The Kubernetes namespace") String namespace,
        @ToolParam(description = "The pod name") String podName
    ) {
        return kubernetesClient.getPodStatus(namespace, podName);
    }
}

ChatClient chatClient = ChatClient.builder(chatModel)
    .defaultTools(new KubernetesTools())
    .build();

String response = chatClient.prompt("Is payments-service healthy in prod?").call().content();
```

The same caveat from the LangChain4j example applies identically here: Spring AI's `@Tool` dispatch calls the annotated method directly once the model requests it, with no built-in confirmation-gate step — any human-in-the-loop requirement (file 4) needs to be implemented inside the method itself, or via a custom advisor intercepting tool execution, which is a more advanced pattern worth exploring once you reach Phase 08's fuller treatment of Spring AI's advisor chain.

## The consistent lesson across both frameworks

Both frameworks solve the same problem this phase's file 1 and Phase 02's file 5 solved by hand — generating a schema from a description and dispatching a call — and solve it well, meaningfully reducing boilerplate. Neither framework, out of the box, enforces this phase's safety disciplines (multi-layer validation, sandboxing, confirmation gating) for you. Those remain engineering decisions you make explicitly, inside your annotated tool methods, regardless of which framework generates the surrounding plumbing. This is worth internalizing as a general pattern for evaluating any framework going forward, not just these two: a framework reducing boilerplate for the *happy path* is not the same as a framework making the *unsafe path* impossible.

## Trade-offs and when this matters most

- For internal tools with low consequence (status checks, read-only queries), either framework's default annotation-driven dispatch is close to sufficient as-is.
- For anything state-changing or irreversible, treat the annotated method as the place where file 3's validation layers and file 4's confirmation gate belong, in either framework — don't assume the framework's convenience wiring implies those protections exist by default.
- Don't pick between LangChain4j's and Spring AI's tool-calling style based on this file alone — Phase 08 covers the fuller comparison, including how each integrates tool calling with memory (Phase 04) and multi-step reasoning (Phase 07), which matters more for a real agent than the annotation syntax alone.

## Why this matters next

You've completed this phase's tool-use discipline, hand-built and framework-packaged. The final project asks you to build a real, safety-conscious tool-using agent against a DevOps-style API, combining schema design, validation, sandboxing, and confirmation gating into one working system — the first project in this handbook where a wrong or unvalidated action could plausibly cause real (simulated) damage if any of this phase's disciplines were skipped.