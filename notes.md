# CONTEXT ENGINEERING — TECHNICAL REFERENCE

**Topics:** LLM System Design · Context Engineering · Retrieval Architectures · Memory Systems · Agentic Systems · Production AI Engineering

**Focus:** Designing reliable, cost-efficient, context-aware LLM applications.

---

# 1. FOUNDATIONS

## 1.1 What Is Context Engineering?

Context Engineering is the discipline of designing, selecting, structuring, managing, and delivering the information an LLM needs at each stage of execution.

It determines:

- **What:** Which information enters the model's context.
- **Source:** Where the information originates.
- **Selection:** Which information is relevant to the current task.
- **Prioritization:** How information is ranked and organized.
- **Timing:** When information is retrieved, updated, or removed.
- **Control:** What information must be excluded.
- **Adaptation:** How context changes throughout execution.

### Primary Objective

Maximize answer quality, task completion, and system reliability while optimizing:

- Token consumption
- Inference cost
- Latency
- Information relevance
- Retrieval quality
- Context utilization
- Operational complexity

**Core Principle:**

Model intelligence alone does not determine application quality. The relevance, completeness, reliability, and structure of supplied context significantly influence model outputs and decisions.

Context Engineering is therefore not simply prompt engineering. It involves designing the complete information pipeline surrounding the model.

## 1.2 LLM Application Architecture

LLM applications can be organized into four broad architectural patterns.

| Architecture | Description | Best Suited For |
|---|---|---|
| LLM Calls | Direct input-output interactions | Generation, classification, extraction |
| Workflows | Predetermined sequences of operations | Predictable, repeatable tasks |
| Agents | Models dynamically select tools and actions | Tasks requiring adaptive decisions |
| Agentic Systems | Coordinate agents, workflows, tools, memory, and execution state | Complex multi-stage objectives |

### Design Principles

1. Prefer deterministic workflows wherever possible.
2. Introduce autonomous decision-making only when the task requires it.
3. Use explicit state transitions to manage execution.
4. Define boundaries between model decisions and deterministic application logic.
5. Add validation, retries, and failure handling around external operations.
6. Keep execution observable and recoverable.

An agent is not automatically better than a workflow. Greater autonomy introduces additional complexity, cost, latency, and failure possibilities.

## 1.3 LLM Limitations

| Limitation | Description |
|---|---|
| Hallucinations | Producing plausible but factually incorrect information |
| Stale Knowledge | Pretrained knowledge may not reflect current information |
| Context Limits | Finite context windows and increasing token costs |
| Lost-in-the-Middle | Relevant information may receive less attention depending on its position |
| Information Integrity | Retrieved information may be incomplete, outdated, contradictory, or unreliable |
| Non-determinism | Identical inputs may produce different outputs |
| Tool Failures | External tools and APIs may fail or return unexpected results |
| Instruction Conflicts | Multiple instructions may contradict each other |
| Safety and Bias | Outputs may contain unsafe or biased information |
| Context Dilution | Excessive irrelevant information can reduce context effectiveness |

**Engineering Implication:**

Production LLM systems must compensate for these limitations through context selection, retrieval validation, structured outputs, evaluation, observability, and controlled execution.

---

# 2. THE SIX SOURCES OF CONTEXT

A production LLM application can assemble information from six major logical sources.

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
- Safety and authorization rules.

**Engineering Consideration:** Instructions should have explicit precedence and conflict-resolution rules. Retrieved documents and tool outputs should be treated as data, not automatically trusted instructions.

## 2.2 External Knowledge

External knowledge supplies information that may not exist in the model's pretrained knowledge or current conversation.

Common sources:

- Vector databases.
- Relational databases.
- Document repositories.
- Search engines.
- External APIs.
- Structured knowledge graphs.

Retrieved information should be evaluated for:

- Relevance.
- Freshness.
- Completeness.
- Source reliability.
- Access permissions.
- Contradictions.

External knowledge is generally retrieved on demand rather than permanently inserting the entire knowledge base into the model's context.

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

A typical tool interaction follows this sequence:

1. The model determines whether a tool is required.
2. The application validates the requested tool and arguments.
3. The tool executes.
4. The application validates the result.
5. The result is added to the next model context.
6. The model continues execution or produces a final response.

Tool execution must account for authorization, timeouts, retries, validation, and side-effect management.

## 2.4 Memory

Memory retains useful information across interactions or tasks.

Examples:

- User preferences.
- Previous decisions.
- Important project information.
- Long-term facts.
- Summaries of previous interactions.

Memory should be selective rather than storing every interaction.

Important memory operations include:

- Extraction: Identify information worth retaining.
- Storage: Persist information in an appropriate representation.
- Retrieval: Find information relevant to the current task.
- Updating: Revise information when new evidence appears.
- Consolidation: Combine related information and remove redundancy.
- Expiration: Remove information that is no longer useful or valid.

Memory retrieval should consider relevance, recency, reliability, and user permissions.

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

State is particularly important for long-running workflows and agentic systems because it enables controlled execution, recovery, and continuation.

**Important Distinction:**

Memory and state serve different purposes:

| Dimension | Memory | State |
|---|---|---|
| Primary Purpose | Retain useful information | Track current execution |
| Lifetime | Across interactions or tasks | During an execution or workflow |
| Examples | Preferences, previous decisions | Current node, retry count |
| Update Pattern | Selective extraction and consolidation | Frequent deterministic updates |
| Storage | Database, vector store, memory service | Checkpoint store, database, execution store |

State should generally be managed by application logic rather than relying exclusively on the LLM to remember its execution progress.

## 2.6 User Prompts

User prompts communicate the immediate intent and requirements of the task.

They may contain:

- Questions.
- Objectives.
- Constraints.
- Preferences.
- Supporting information.
- Expected output formats.
- Additional instructions.

The application should distinguish between the user's current request and previously stored preferences or instructions.

For example, a user's current explicit instruction may override an older preference, subject to higher-priority system and developer constraints.

---

# 3. CONTEXT ASSEMBLY

Context Assembly is the process of transforming information from different sources into a structured input that the model can use for a particular execution step.

It is one of the central responsibilities of a production context engineering system.

## 3.1 Context Assembly Pipeline

A typical pipeline consists of:

1. **Input Processing:** Understand the current task and extract requirements.
2. **Source Identification:** Determine which information sources are required.
3. **Retrieval:** Fetch relevant documents, memories, and external data.
4. **Filtering:** Remove irrelevant, duplicate, unauthorized, or unreliable information.
5. **Ranking:** Prioritize information based on task relevance and reliability.
6. **Compression:** Reduce unnecessary tokens while preserving important meaning.
7. **Assembly:** Combine instructions, knowledge, memory, state, and user input.
8. **Validation:** Check token budgets, required fields, permissions, and conflicts.
9. **Delivery:** Supply the assembled context to the LLM.

## 3.2 Context Prioritization

Not every available piece of information deserves equal priority.

A context selection strategy should consider:

| Factor | Purpose |
|---|---|
| Relevance | How closely information matches the current task |
| Authority | Whether the source is trustworthy |
| Recency | Whether the information is still current |
| Reliability | How confident the system is in the information |
| Importance | How significantly it affects the decision |
| Permissions | Whether the information can be accessed and used |

**Engineering Principle:** Relevance alone is insufficient. A highly relevant but outdated or unauthorized document should not automatically enter the context.

## 3.3 Context Budget Management

LLMs operate within finite context windows. Context assembly must therefore allocate tokens deliberately.

A conceptual budget:

\[
T_{total} = T_{instructions} + T_{history} + T_{retrieval}
\]

\[
+ T_{memory} + T_{state} + T_{tools} + T_{output}
\]

Where:

- \(T_{instructions}\): Instruction tokens.
- \(T_{history}\): Relevant conversation history.
- \(T_{retrieval}\): Retrieved knowledge.
- \(T_{memory}\): Retrieved memory.
- \(T_{state}\): Execution state.
- \(T_{tools}\): Tool descriptions and results.
- \(T_{output}\): Reserved output tokens.

The total must remain within the model's effective context limit.

Context budgets should be dynamic rather than allocating a fixed number of tokens to every source.

---

# 4. RETRIEVAL ARCHITECTURES

Retrieval provides external information to an LLM when the required knowledge is not already available in its context.

## 4.1 Retrieval-Augmented Generation (RAG)

RAG combines information retrieval with LLM generation.

### Typical RAG Pipeline

1. Document ingestion.
2. Text extraction and cleaning.
3. Document chunking.
4. Embedding generation.
5. Vector database storage.
6. User query processing.
7. Similarity-based retrieval.
8. Context construction.
9. LLM generation.
10. Response evaluation.

## 4.2 Retrieval Strategies

| Strategy | Description | Use Case |
|---|---|---|
| Dense Retrieval | Embedding-based semantic search | Conceptual and semantic queries |
| Sparse Retrieval | Keyword-based retrieval such as BM25 | Exact terms and identifiers |
| Hybrid Retrieval | Combines dense and sparse retrieval | General-purpose knowledge retrieval |
| Metadata Filtering | Restricts retrieval using structured attributes | Tenant, date, document type |
| Reranking | Reorders retrieved results using a relevance model | Improving top-k relevance |
| Query Rewriting | Transforms the original query for better retrieval | Ambiguous or complex queries |
| Multi-Query Retrieval | Generates multiple search queries | Broader information coverage |

## 4.3 Advanced Retrieval Techniques

**Query Decomposition:** Break complex questions into smaller retrieval tasks.

**Contextual Compression:** Extract only the relevant portions of retrieved documents.

**Parent-Child Retrieval:** Retrieve small chunks for precision while preserving access to their larger document context.

**Recursive Retrieval:** Perform additional retrieval based on information discovered during an earlier retrieval stage.

**Corrective Retrieval:** Detect insufficient or unreliable results and trigger alternative retrieval strategies.

**Engineering Principle:** Retrieval quality should be evaluated independently from generation quality. A model cannot reliably answer using information that the retrieval system failed to provide.

---

# 5. MEMORY ARCHITECTURES

Memory systems allow applications to retain and reuse information beyond an individual model call.

## 5.1 Memory Types

| Memory Type | Description | Examples |
|---|---|---|
| Working Memory | Information required for the current task | Current context, intermediate results |
| Short-Term Memory | Information retained within a limited interaction or session | Recent conversation history |
| Long-Term Memory | Information retained across sessions | User preferences, persistent facts |
| Episodic Memory | Records of previous experiences or events | Previous tasks and their outcomes |
| Semantic Memory | Generalized facts and knowledge | Stable preferences, learned concepts |
| Procedural Memory | Knowledge about how to perform tasks | Reusable workflows and operating procedures |

These categories are conceptual and can overlap. They do not necessarily require separate databases.

## 5.2 Memory Lifecycle

A production memory system should implement:

1. **Extraction:** Identify potentially useful information.
2. **Qualification:** Determine whether it is worth retaining.
3. **Storage:** Save it with appropriate metadata.
4. **Retrieval:** Select relevant memories for the current task.
5. **Updating:** Modify outdated or corrected information.
6. **Consolidation:** Merge duplicates and related information.
7. **Expiration:** Apply retention policies.
8. **Deletion:** Respect user requests and data retention requirements.

## 5.3 Memory Retrieval

Memory retrieval can combine:

- Semantic similarity.
- Keyword matching.
- Recency.
- Importance.
- User or session identity.
- Metadata filtering.
- Access permissions.

A conceptual ranking function:

\[
Score(m,q) =
w_rR(m,q) + w_iI(m) + w_tT(m)
\]

Where:

- \(R(m,q)\): Relevance to the current query.
- \(I(m)\): Importance of the memory.
- \(T(m)\): Temporal relevance.
- \(w_r,w_i,w_t\): Tunable weights.

This is an illustrative ranking model, not a universal scoring formula.

**Engineering Principle:** Memory quality depends on what the system stores, what it retrieves, when it retrieves it, and whether it correctly updates old information.

---

# 6. CONTEXT MANAGEMENT IN AGENTIC SYSTEMS

Context requirements change throughout agent execution.

An agent may need different information when planning, selecting tools, executing operations, validating results, and producing its final response.

## 6.1 Context Changes During Execution

A typical agent execution cycle:

1. Receive the user request.
2. Assemble initial context.
3. Analyze the task.
4. Select an action or tool.
5. Execute the action.
6. Process the tool result.
7. Update execution state.
8. Retrieve additional context if required.
9. Continue or terminate execution.
10. Produce the final response.

## 6.2 Context Management Strategies

| Strategy | Purpose |
|---|---|
| Context Pruning | Remove irrelevant information |
| Context Compression | Reduce token usage while retaining meaning |
| Context Summarization | Represent lengthy history in a shorter form |
| Just-in-Time Retrieval | Retrieve information only when needed |
| Context Isolation | Separate information between users, tenants, or tasks |
| Context Refreshing | Replace stale information with updated information |
| Context Checkpointing | Preserve information required to resume execution |

## 6.3 Long-Running Agent Execution

Long-running workflows require more than conversation history.

They may need:

- Persistent execution state.
- Checkpoint storage.
- Task identifiers.
- Retry policies.
- Idempotency controls.
- Tool execution records.
- Human approval checkpoints.
- Recovery mechanisms.

**Important:** A checkpoint should preserve the information needed to resume execution safely. It should not automatically preserve every token from the previous model context.

---

# 7. CONTEXT EVALUATION AND OBSERVABILITY

Context engineering must be measurable. A context pipeline that appears effective in a demonstration may fail under real-world workloads.

## 7.1 Evaluation Dimensions

| Dimension | What to Measure |
|---|---|
| Retrieval Quality | Recall@K, Precision@K, MRR, NDCG |
| Context Relevance | Relevance of supplied information to the task |
| Context Completeness | Whether essential information was included |
| Groundedness | Whether outputs are supported by available evidence |
| Task Success | Whether the application completed the intended task |
| Token Efficiency | Tokens consumed per successful task |
| Latency | Retrieval, assembly, and generation latency |
| Cost | Cost per execution and successful task |
| Reliability | Failure rate, timeout rate, and recovery success |

## 7.2 Context-Level Testing

Test the context pipeline independently from the final LLM output.

Examples:

- Did retrieval return the expected documents?
- Were unauthorized documents excluded?
- Was relevant memory retrieved?
- Was stale information filtered?
- Were conflicting facts detected?
- Did context compression preserve important details?
- Did token-budget enforcement work correctly?
- Did tenant isolation prevent cross-user information leakage?

## 7.3 Observability

Production systems should capture:

- Context assembly duration.
- Retrieval latency.
- Retrieved document identifiers.
- Memory retrieval results.
- Prompt and context version.
- Input and output token counts.
- Model latency and cost.
- Tool execution traces.
- Retry and failure counts.
- Evaluation results.

Sensitive context should not be indiscriminately logged. Apply redaction, access controls, and retention policies.

## 7.4 Evaluation Harness

A practical evaluation harness consists of three major components:

| Component | Responsibility |
|---|---|
| Evaluator | Measures context and output quality |
| Runner | Executes test cases against the system |
| Reporter | Aggregates results and identifies regressions |

Evaluation should cover both individual components and the complete application.

Recommended testing layers:

1. Retrieval component evaluations.
2. Memory retrieval and update evaluations.
3. Context assembly tests.
4. Tool execution tests.
5. End-to-end task evaluations.
6. Regression evaluations.

---

# 8. PRODUCTION ENGINEERING PRINCIPLES

Context engineering is not only an LLM concern. It intersects with backend engineering, distributed systems, security, and reliability.

### 8.1 Security

- Enforce tenant and user isolation.
- Validate tool permissions before execution.
- Treat retrieved content as untrusted data.
- Defend against indirect prompt injection.
- Prevent unauthorized memory access.
- Apply data retention and deletion policies.
- Redact sensitive information from logs.

### 8.2 Reliability

- Use bounded retries.
- Validate structured outputs.
- Implement timeouts.
- Persist critical execution state.
- Use idempotency for side-effecting operations.
- Implement checkpoint recovery.
- Fail safely when required context is unavailable.

### 8.3 Performance and Cost

- Retrieve only relevant information.
- Use metadata filtering before expensive retrieval operations.
- Cache appropriate retrieval results.
- Compress context selectively.
- Monitor token consumption.
- Avoid unnecessary model calls.
- Use smaller models for suitable classification and extraction tasks.

### 8.4 Architectural Separation

Keep these responsibilities clearly separated:

- Context retrieval.
- Memory management.
- Context assembly.
- Model inference.
- Tool execution.
- State persistence.
- Evaluation.
- Observability.

This separation makes individual components easier to test, optimize, replace, and debug.

---

# 9. FINAL ENGINEERING MODEL

A useful mental model for production Context Engineering is:

**Context Sources → Context Retrieval → Context Selection → Context Assembly → LLM Execution → Result Validation → State and Memory Updates → Evaluation**

This is a logical architecture, not a mandatory linear execution sequence. Production systems may repeat retrieval, assembly, and execution as workflows progress.

### Core Takeaways

1. Context engineering is broader than prompt engineering.
2. Context must be selected, not simply accumulated.
3. Memory and execution state are different architectural responsibilities.
4. Retrieval quality directly influences downstream generation quality.
5. Context budgets must account for cost, latency, and model limits.
6. Agents require dynamic context management throughout execution.
7. Context pipelines need independent evaluation and observability.
8. Security and tenant isolation must be enforced before information reaches the model.
9. Deterministic application logic should control critical operations.
10. Production reliability comes from the complete system, not from the model alone.

**Final Principle:**

A reliable LLM application does not simply provide more context. It provides the right information, from the right source, at the right time, in the right structure, with the right controls.