# CONTEXT ENGINEERING — TECHNICAL REFERENCE

**Topics:** LLM System Design · Context Engineering · Retrieval Architectures · Memory Systems · Agentic Systems · Production AI Engineering

---

# 1. FOUNDATION

## 1.1 What is Context Engineering?

Context Engineering is the discipline of designing, selecting, structuring, managing, and delivering the information an LLM needs at each stage of execution.

It determines:

- **What:** Which information enters the model's context.
- **Source:** Where the information originates.
- **Selection:** Which information is relevant to the current task.
- **Prioritization:** How information is ranked and organized.
- **Timing:** When information is retrieved, updated, or removed.
- **Control:** What information must be excluded.
- **Adaptation:** How context changes throughout execution.

**Primary Objective:**

Maximize answer quality, task completion, and system reliability while optimizing:

- Token consumption
- Inference cost
- Latency
- Information relevance
- Context utilization
- Retrieval quality
- Operational complexity

**Core Principle:**

Model intelligence alone does not determine application quality. The quality, relevance, completeness, and structure of the supplied context significantly influence the model's output and decisions.

Context engineering is therefore not simply prompt engineering. It involves designing the complete information pipeline surrounding the model.

## 1.2 LLM System Architecture

LLM applications can be organized into four broad architectural patterns.

| Architecture | Description | Best Suited For |
|---|---|---|
| LLM Calls | Direct input-output interactions | Generation, classification, extraction |
| Workflows | Predetermined sequences of operations | Predictable and repeatable tasks |
| Agents | Models dynamically select tools and actions | Tasks requiring adaptive decisions |
| Agentic Systems | Coordinate agents, workflows, tools, memory and execution state | Complex multi-stage objectives |

**Design Principles:**

1. Prefer deterministic workflows wherever possible.
2. Introduce autonomous decision-making only when the task requires it.
3. Use explicit state transitions to manage execution.
4. Define boundaries between model decisions and deterministic application logic.
5. Add validation, retries, and failure handling around external operations.

An agent is not automatically better than a workflow. Greater autonomy introduces additional complexity, cost, latency, and failure possibilities.

## 1.3 LLM Limitations

| Limitation | Description |
|---|---|
| Hallucinations | Producing plausible but factually incorrect information |
| Stale Knowledge | Pretrained knowledge may not reflect current information |
| Context Limits | Finite context windows and increasing token costs |
| Lost-in-the-Middle | Relevant information may receive less attention depending on its position within the context |
| Information Integrity | Retrieved information may be incomplete, outdated, contradictory, or unreliable |
| Non-determinism | Identical inputs may produce different outputs |
| Tool Failures | External tools and APIs may fail or return unexpected results |
| Instruction Conflicts | Multiple instructions may contradict each other |
| Safety and Bias | Outputs may contain unsafe, biased, or policy-violating content |
| Context Dilution | Excessive irrelevant information can reduce the effectiveness of the supplied context |

**Engineering Implication:**

Production LLM systems must compensate for these limitations through context selection, retrieval validation, structured outputs, evaluation, observability, and controlled execution.

---

# 2. THE SIX SOURCES OF CONTEXT

Context is not limited to retrieved documents or conversation history. A production LLM application can assemble information from six major sources.

| Source | Purpose | Examples |
|---|---|---|
| Instructions | Define model behavior and constraints | System prompts, policies, task instructions, output schemas |
| External Knowledge | Supply factual information | Documents, databases, APIs, knowledge bases |
| Tools | Enable external computation and actions | Python, web search, APIs, database queries |
| Memory | Retain useful information across interactions | User preferences, previous decisions, learned facts |
| State | Track current execution and progress | Workflow step, retry count, pending tasks, execution status |
| User Prompts | Express user intent and requirements | Questions, goals, constraints, additional instructions |

**Important:** These are logical sources, not necessarily six separate components. An application may combine them into a single context assembly pipeline.

## 2.1 Instructions

Instructions establish the behavioral and operational boundaries of the model.

They define:

- What the model should accomplish.
- Which rules it must follow.
- Which actions it can perform.
- Which constraints it must respect.
- How outputs should be structured.

Examples:

- System instructions.
- Developer instructions.
- Task-specific instructions.
- JSON output schemas.
- Tool usage policies.

## 2.2 External Knowledge

External knowledge supplies information that may not exist in the model's pretrained knowledge or current conversation.

Common sources:

- Vector databases.
- Relational databases.
- Document repositories.
- Search engines.
- External APIs.
- Structured knowledge graphs.

The retrieved information should be evaluated for relevance, freshness, completeness, and source reliability before being supplied to the model.

## 2.3 Tools

Tools extend the capabilities of an LLM application beyond text generation.

Examples:

- Python execution.
- Web search.
- Database queries.
- File operations.
- External API calls.
- Code execution.

**Important Distinction:**

A tool is an available capability. Its execution result becomes information that can enter the model's subsequent context.

Tool execution must also account for authorization, timeouts, retries, validation, and side-effect management.

## 2.4 Memory

Memory retains useful information across interactions or tasks.

Examples:

- User preferences.
- Previous decisions.
- Important project information.
- Long-term facts.
- Summaries of previous interactions.

Memory should be selective rather than storing every interaction. Information should be retrieved when relevant and updated when new evidence changes previously stored information.

## 2.5 State

State describes the current condition and progress of an execution.

Examples:

- Current workflow step.
- Execution status.
- Retry count.
- Pending actions.
- Tool execution results.
- Checkpoint information.
- Human approval status.

State is particularly important for long-running workflows and agentic systems because it enables controlled execution,