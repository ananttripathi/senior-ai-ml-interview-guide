# Agentic AI Systems

The fastest-growing interview topic at MAANG as of 2025–2026. Missing from all pre-2024 resources. Now tested explicitly at Google DeepMind, Meta AI, Amazon, and AI-native startups.

---

## What is an AI Agent?

An AI agent is an LLM that can:
1. **Perceive** its environment (tool outputs, user input, memory)
2. **Reason** about what action to take next
3. **Act** by calling tools, writing to memory, or producing output
4. **Iterate** — repeat the loop until the task is complete

Agents differ from plain LLM calls by having a **feedback loop**: the model observes the result of its actions and adjusts.

---

## ReAct Framework (Reasoning + Acting)

The foundational pattern for agents. Each step interleaves reasoning and action:

```
Thought: I need to find the weather in Tokyo.
Action: search("current weather Tokyo")
Observation: Tokyo: 22°C, partly cloudy
Thought: I have the weather. I can now answer the user.
Action: finish("It's 22°C and partly cloudy in Tokyo.")
```

**Why it works**: forcing explicit reasoning before each action reduces hallucinated actions and improves tool selection accuracy.

**Failure modes**: the model can get stuck in reasoning loops ("Thought: I need to think more..."), or take too many unnecessary steps.

---

## Plan-and-Execute Architecture

For long-horizon tasks, separate planning from execution:

```
Step 1: Planner LLM → generates a high-level plan (list of subtasks)
Step 2: Executor agent → executes each subtask using tools
Step 3: Replanner → evaluates results, revises remaining plan if needed
```

**Advantage over ReAct**: better for complex multi-step tasks where the full plan should be visible upfront.
**Disadvantage**: less adaptive — plan can become stale as new information arrives.

---

## Tool Use / Function Calling

Modern LLMs can select and call external tools:

```json
// Model outputs structured tool call
{
  "tool": "search_web",
  "arguments": {"query": "LLaMA 3 benchmark results"}
}

// System executes tool, returns result
{
  "result": "LLaMA 3 70B scores 82.0 on MMLU..."
}
```

**Function calling** (OpenAI/Anthropic): tools are declared as JSON schemas; model outputs structured JSON rather than free text. More reliable than parsing tool calls from unstructured output.

### Common tool categories
| Category | Examples |
|----------|---------|
| Search/retrieval | Web search, vector DB lookup, SQL query |
| Code execution | Python REPL, bash, Jupyter |
| API calls | REST APIs, calendar, email, Slack |
| File I/O | Read/write files, upload, download |
| Browser | Selenium/Playwright for web automation |

### Tool design principles
- **Idempotent where possible**: tools that can be safely retried on failure
- **Atomic**: one tool does one thing — avoid multi-action tools
- **Well-typed**: clear input/output schemas reduce model errors
- **Include examples in descriptions**: the model uses tool descriptions to decide when to call them

---

## Memory Types

Agents need different types of memory for different purposes:

| Type | Analogy | Implementation | Persistence |
|------|---------|---------------|-------------|
| **Working** | RAM | Context window (in-prompt) | Current session only |
| **Episodic** | Diary | Vector DB (retrieve by similarity) | Cross-session |
| **Semantic** | Knowledge base | Knowledge graph / structured DB | Long-term facts |
| **Procedural** | Muscle memory | Fine-tuned weights / system prompt | Baked into model |

### Working memory limits
Context window is finite. Strategies for long tasks:
- **Summarization**: compress old conversation turns into a summary
- **Retrieval-augmented memory**: store detailed history in vector DB, retrieve relevant chunks
- **Sliding window**: drop oldest messages when context fills

---

## Multi-Agent Orchestration

Single agents work well for simple tasks. Multi-agent systems handle complexity via specialization:

### Patterns

**Hierarchical (Manager-Worker)**
```
Orchestrator Agent
  ├── Research Agent  (web search, document analysis)
  ├── Coding Agent    (writes and tests code)
  └── Writer Agent    (formats and summarizes output)
```
Orchestrator breaks the task, delegates to specialists, aggregates results.

**Collaborative (Peer-to-Peer)**
Agents share a message bus; any agent can produce tasks that others pick up.

**Debate / Critique**
Two agents generate competing answers; a judge agent selects the better one. Improves accuracy on complex reasoning tasks.

### Frameworks
| Framework | Pattern | Best for |
|-----------|---------|---------|
| LangGraph | Graph-based state machine | Complex workflows with branching |
| CrewAI | Role-based teams | Structured multi-agent pipelines |
| AutoGen | Conversational agents | Research, code generation |
| Semantic Kernel | Enterprise orchestration | .NET/Java enterprise apps |

---

## Model Context Protocol (MCP)

MCP (Anthropic, 2024) is an open standard for connecting AI models to external tools and data sources. Think of it as "USB-C for AI tools."

**Without MCP**: each application builds custom integrations for each tool. N applications × M tools = N×M integrations.

**With MCP**: each tool exposes an MCP server; any MCP-compatible model can use any tool. N + M connections total.

### Architecture
```
Claude / GPT / Gemini
        ↕  MCP protocol
   MCP Client (in your app)
        ↕
   MCP Server (for each tool)
     ├── file_system_server
     ├── github_server
     ├── postgres_server
     └── web_search_server
```

### MCP primitives
- **Resources**: data the server exposes (files, DB rows, API responses)
- **Tools**: functions the model can call
- **Prompts**: reusable prompt templates served by the server

**Interview angle**: "How would you architect tool access for an enterprise AI assistant serving 10,000 users?"
Answer: MCP servers for each internal tool (CRM, HR, ticketing); auth layer in front of each MCP server; rate limiting and audit logging at the MCP client layer; capability-based access control per user role.

---

## Guardrails and Safety

Production agents need guardrails at multiple layers:

### Input guardrails
- **Prompt injection detection**: classify user input for injection attempts ("ignore previous instructions")
- **PII detection**: identify and mask personal data before sending to LLM
- **Content policy filter**: block disallowed request types

### Output guardrails
- **Hallucination detection**: compare LLM output to retrieved sources; flag low-confidence claims
- **Schema validation**: if output should be JSON, validate before returning
- **Toxicity / harmful content filter**: run output through classifier
- **Sensitive action confirmation**: require user confirmation before irreversible actions (emails, file deletion, payments)

### Action guardrails
- **Dry-run mode**: for write operations, preview action before executing
- **Rate limiting**: cap tool calls per session to prevent runaway loops
- **Reversibility check**: categorize tools by reversibility; require explicit approval for irreversible

---

## Agentic System Design — Interview Framework

When asked "Design an AI agent for X":

### 1. Task decomposition
- What subtasks does X require?
- Which subtasks benefit from LLM reasoning vs deterministic code?

### 2. Tool definition
- What tools does the agent need? Define inputs/outputs for each.
- Which tools are read-only (safe) vs write (need guardrails)?

### 3. Memory architecture
- What state needs to persist across steps? Across sessions?
- How do you prevent context overflow for long tasks?

### 4. Orchestration pattern
- Single agent with tools, or multiple specialized agents?
- How do agents hand off work?

### 5. Error handling
- What happens when a tool fails? Retry strategy?
- What happens when the agent loops or gets stuck? Max iterations limit?
- Human-in-the-loop: when should the agent stop and ask for clarification?

### 6. Evaluation
- Task completion rate (did it complete the task?)
- Tool call accuracy (did it call the right tool with the right arguments?)
- Step efficiency (did it take the minimum steps needed?)
- Safety (did it ever take unauthorized actions?)

---

## Common Interview Questions

**"Design a coding assistant agent that can write, test, and debug code."**

Key components:
- Tools: code_write(file, content), code_run(file) → stdout/stderr, file_read(file), search_docs(query)
- Loop: write → run → observe output → fix if error → run again
- Guardrails: sandbox execution (Docker), max retries, timeout per run
- Evaluation: does the code pass the test suite? Measure pass@k (fraction of tasks solved in k attempts)

**"What's the difference between a chain and an agent?"**
- **Chain**: fixed sequence of LLM calls and tool calls; deterministic flow
- **Agent**: the LLM dynamically decides which tool to call next based on observations; non-deterministic flow

Use chains when the workflow is well-defined and predictable. Use agents when the task requires adaptive decision-making.

**"How do you prevent an agent from running infinitely?"**
- Hard cap on number of iterations (e.g., max 25 steps)
- Timeout at the task level
- Detect repetition: if the same tool is called with the same arguments 3 times, force termination
- Budget awareness: track token/API cost; stop if budget exceeded

**"How do you evaluate a multi-agent system?"**
- End-to-end task success rate (did the user's goal get achieved?)
- Per-agent metrics (did each specialist complete its subtask correctly?)
- Handoff quality (did the orchestrator's decomposition make sense?)
- Latency and cost breakdown by agent
- Human evaluation on a golden test set (especially for open-ended tasks)

---

## Token Management & Cost Optimization

At scale, agent token cost is significant:

| Strategy | Description |
|----------|------------|
| Smaller models for simple tasks | Route tool-selection to a cheaper model; use frontier model only for reasoning |
| Caching | Cache identical tool results; cache LLM responses for repeated subtasks |
| Structured outputs | JSON mode reduces output tokens vs prose; easier to parse |
| Context pruning | Summarize or truncate old reasoning steps that are no longer needed |
| Batching | For parallel tool calls, batch into one LLM call with multiple function calls |

---

## Key Terms Cheat Sheet

| Term | Definition |
|------|-----------|
| ReAct | Reasoning + Acting pattern; interleaves Thought/Action/Observation |
| Tool use / function calling | Structured LLM output that triggers external function execution |
| MCP | Model Context Protocol — open standard for AI ↔ tool connections |
| Orchestrator | Top-level agent that decomposes tasks and delegates to sub-agents |
| Prompt injection | Adversarial input designed to override system instructions |
| HITL | Human-in-the-loop — agent pauses and asks human to confirm before acting |
| Pass@k | Evaluation metric: fraction of problems solved correctly in k attempts |
| Hallucination grounding | Technique to anchor LLM output in retrieved evidence |
