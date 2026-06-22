# 🤖 Agentic AI for Full Stack Developers

> Modern full-stack developers are increasingly expected to build AI-powered features — from simple completions to autonomous agents. This section covers the core concepts interviewers are starting to ask about.

---

### Q35. What is an AI Agent and how is it different from a simple LLM call?

| | LLM Call | AI Agent |
|---|---|---|
| Input/Output | Prompt → Text | Goal → Actions → Result |
| Memory | None (stateless) | Short-term + long-term |
| Tools | ❌ No | ✅ Yes (search, DB, APIs) |
| Autonomy | Single turn | Multi-step reasoning |

```csharp
// Simple LLM call — one shot
var response = await client.GetChatMessageContentsAsync("Summarize this text...");

// Agent — multi-step, uses tools, remembers context
var agent = new ChatCompletionAgent()
{
    Name = "ResearchAgent",
    Instructions = "You help users research topics using available tools.",
    Kernel = kernel // kernel has tools registered
};
```

---

### Q36. What is Semantic Kernel and why is it used in .NET agentic apps?

Semantic Kernel is Microsoft's open-source SDK for building AI agents and copilots in .NET (and Python/Java).

```csharp
// Install: dotnet add package Microsoft.SemanticKernel

var builder = Kernel.CreateBuilder();
builder.AddAzureOpenAIChatCompletion(
    deploymentName: "gpt-4",
    endpoint: config["AzureOpenAI:Endpoint"],
    apiKey: config["AzureOpenAI:ApiKey"]
);

var kernel = builder.Build();

var result = await kernel.InvokePromptAsync(
    "Translate {{$input}} to French",
    new KernelArguments { ["input"] = "Hello World" }
);
```

---

### Q37. What is a Kernel Plugin (Tool) in Semantic Kernel?

Plugins expose native C# methods as tools the AI can call during reasoning.

```csharp
public class OrderPlugin
{
    [KernelFunction, Description("Get order status by order ID")]
    public async Task<string> GetOrderStatus(
        [Description("The order ID")] string orderId)
    {
        return $"Order {orderId} is shipped and will arrive tomorrow.";
    }

    [KernelFunction, Description("Cancel an order")]
    public async Task<string> CancelOrder(string orderId)
    {
        return $"Order {orderId} has been cancelled.";
    }
}

// Register plugin — AI will auto-call these when relevant
kernel.Plugins.AddFromType<OrderPlugin>();
```

---

### Q38. What is Function Calling / Tool Calling in AI APIs?

Function calling lets the model decide when to invoke your code, passing structured arguments.

```csharp
// Flow:
// 1. User: "What's the status of order #1234?"
// 2. Model: [calls GetOrderStatus("1234")]
// 3. Your code runs, returns result
// 4. Model: "Your order #1234 has shipped and arrives tomorrow."

// Manual definition with Azure OpenAI SDK:
var tools = new List<ChatCompletionsFunctionToolDefinition>
{
    new("GetOrderStatus", "Get order status", BinaryData.FromString("""
    {
        "type": "object",
        "properties": {
            "orderId": { "type": "string", "description": "The order ID" }
        },
        "required": ["orderId"]
    }
    """))
};
```

---

### Q39. What are the types of memory in an AI Agent?

```
┌─────────────────┬──────────────────────────────┬─────────────────────────┐
│ Type            │ What it stores               │ .NET implementation     │
├─────────────────┼──────────────────────────────┼─────────────────────────┤
│ In-context      │ Chat history in prompt        │ ChatHistory object      │
│ Semantic (RAG)  │ Vector embeddings in DB       │ Qdrant, Azure AI Search │
│ Episodic        │ Past interactions/summaries   │ DB + retrieval plugin   │
│ Procedural      │ Instructions / system prompt  │ Agent Instructions      │
└─────────────────┴──────────────────────────────┴─────────────────────────┘
```

```csharp
// Managing chat history (in-context memory)
var history = new ChatHistory();
history.AddSystemMessage("You are a helpful .NET assistant.");
history.AddUserMessage("What is dependency injection?");

var response = await chatService.GetChatMessageContentAsync(history);
history.AddAssistantMessage(response.Content);
```

---

### Q40. What is RAG (Retrieval Augmented Generation)?

RAG enriches the LLM prompt with relevant documents fetched from a vector store — instead of fine-tuning the model.

```
Flow:
User Query → Embed Query → Search Vector DB → Retrieve Top-K Docs
→ Inject into Prompt → LLM generates grounded answer
```

```csharp
var memory = new SemanticTextMemory(azureSearchStore, embeddingGenerator);

// Save document chunks
await memory.SaveInformationAsync(
    collection: "products",
    text: "Product X has a 2-year warranty and supports 4K output.",
    id: "product-x-warranty"
);

// Retrieve at query time and inject into prompt
var results = memory.SearchAsync("warranty policy", collection: "products", limit: 3);
await foreach (var result in results)
    context += result.Metadata.Text + "\n";
```

---

### Q41. What is an Agentic Loop / ReAct pattern?

ReAct (Reasoning + Acting) is the pattern most AI agents follow internally.

```
1. THINK   — model reasons about the goal
2. ACT     — model calls a tool
3. OBSERVE — tool result returned to model
4. REPEAT  — until goal achieved or max steps hit
```

```csharp
// Semantic Kernel handles the loop automatically
var settings = new OpenAIPromptExecutionSettings
{
    FunctionChoiceBehavior = FunctionChoiceBehavior.Auto()
};

// Will auto-call: CheckCalendar → CreateMeeting → SendEmail
var result = await kernel.InvokePromptAsync(
    "Book a meeting with John for tomorrow at 10am and send him a confirmation.",
    new KernelArguments(settings)
);
```

---

### Q42. How do you stream responses from an LLM in ASP.NET Core?

```csharp
// Server-Sent Events (SSE) endpoint
[HttpGet("stream")]
public async IAsyncEnumerable<string> StreamChat(string prompt)
{
    await foreach (var chunk in kernel.InvokePromptStreamingAsync(prompt))
        yield return chunk.ToString();
}

// Frontend — EventSource API
const source = new EventSource('/api/chat/stream?prompt=Hello');
source.onmessage = (e) => { document.body.innerHTML += e.data; };
```

---

### Q43. What are the key risks when building agentic systems?

```
Risk                     │ Mitigation
─────────────────────────┼──────────────────────────────────────────
Prompt injection         │ Validate & sanitize all tool inputs
Runaway tool calls       │ Set max iteration limit (e.g. 10 steps)
Sensitive data leakage   │ Redact PII before sending to LLM
Hallucinated tool args   │ Validate args in plugin before executing
Cost overrun             │ Track token usage, set budget alerts
```

```csharp
var thread = new AgentGroupChat();
thread.ExecutionSettings = new AgentGroupChatSettings
{
    TerminationStrategy = new DefaultTerminationStrategy { MaximumIterations = 10 }
};
```

---

### Q44. What is Multi-Agent architecture?

Multiple specialized agents collaborate, each owning a domain — orchestrated by a supervisor agent.

```
                   ┌──────────────────┐
                   │ Supervisor Agent  │
                   └────────┬─────────┘
          ┌─────────────────┼──────────────────┐
   ┌──────▼──────┐  ┌───────▼───────┐  ┌───────▼──────┐
   │  Research   │  │  Code Writer  │  │  QA Reviewer │
   │  Agent      │  │  Agent        │  │  Agent       │
   └─────────────┘  └───────────────┘  └──────────────┘
```

```csharp
var groupChat = new AgentGroupChat(supervisor, researcher, writer)
{
    ExecutionSettings = new AgentGroupChatSettings
    {
        SelectionStrategy = new KernelFunctionSelectionStrategy(selectionFn, kernel),
        TerminationStrategy = new KernelFunctionTerminationStrategy(terminateFn, kernel)
    }
};
```

---

## ☁️ Azure for .NET Developers

> **Coming next week** — this section is underway and will cover App Service, Azure Functions, Service Bus, Key Vault, Managed Identity, Application Insights, APIM, and CI/CD with GitHub Actions.

---

*Part of [.NET & C# Interview Questions](./README.md) — maintained by [Hemant Gupta](https://in.linkedin.com/in/hemantguptaa)*
