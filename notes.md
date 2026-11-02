# CONTEXT ENGINEERING — TECHNICAL REFERENCE

**Topics:** LLM System Design · Context Engineering · Retrieval Architectures · Memory Systems · Agentic Systems · Production AI Engineering

---

# 1. FOUNDATION

## 1.1 What is Context Engineering?

Context Engineering is the discipline of designing, selecting, structuring, managing, and delivering the information an LLM needs at each stage of execution.

It determines:
- What information enters the model's context.
- Where that information comes from.
- How information is prioritized and organized.
- When information is retrieved or updated.
- What information should be excluded.
- How context changes throughout execution.

**Primary Objective:**

Maximize answer quality and system reliability while optimizing:

- Token consumption
- Latency
- Inference cost
- Information relevance
- Context utilization
- Operational complexity

**Core Principle:** Model intelligence alone does not determine system quality. The quality, relevance, and structure of the supplied context significantly influence the final result.

## 1.2 LLM System Architecture

LLM applications can be classified into four broad categories.

| Category | Description | Best Suited For |
|---|---|---|
| LLM Calls | Direct input-output interactions | Simple generation and classification |
| Workflows | Predetermined sequences of operations | Predictable, repeatable tasks |
| Agents | Dynamically select tools and actions | Tasks requiring adaptive decisions |
| Agentic Systems | Coordinate agents, workflows, tools and memory | Complex multi-stage objectives |

**Design Principle:**

Prefer deterministic workflows wherever possible. Introduce autonomous decision-making only when the task genuinely requires it.

An agent is not automatically better than a workflow.

## 1.3 LLM Limitations

| Limitation | Description |
|---|---|
| Hallucinations | Producing plausible but factually incorrect information |
| Stale Knowledge | Pretrained knowledge may not reflect current information |
| Context Limits | Finite context windows and increasing token costs |
| Lost-in-the-Middle | Relevant information can receive less attention depending on its position |
| Information Integrity | Retrieved information may be incomplete, outdated or contradictory |
| Non-determinism | Identical inputs may produce different outputs |
| Tool Failures | External tools and APIs may fail or return unexpected results |
| Safety and Bias | Outputs may contain unsafe or biased information |

---

# 2. THE SIX SOURCES OF CONTEXT

Context is not limited to retrieved documents. It consists of six major information sources.

| Source | Purpose | Examples |
|---|---|---|
| Instructions | Define behavior and constraints | System prompts, policies, schemas |
| External Knowledge | Supply factual information | Documents, databases, APIs |
| Tools | Enable external actions | Python, web search, APIs |
| Memory | Retain useful information | Preferences, previous decisions |
| State | Track execution status | Workflow step, retries, pending tasks |
| User Prompts | Express user intent | Questions, goals, constraints |

## 2.1 Important Distinctions

**Memory vs State vs External Knowledge**

- Memory: Information retained across interactions or tasks.
- State: Information describing the current execution and its progress.
- External Knowledge: Information retrieved from external sources.

Example:

A research agent investigating a company:

- Memory: User prefers financial analysis using primary sources.
- State: Currently investigating the company's quarterly results.
- External Knowledge: Latest annual report and financial statements.

