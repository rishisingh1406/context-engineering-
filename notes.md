# CONTEXT ENGINEERING — TECHNICAL REFERENCE

**Topics:** LLM System Design · Context Engineering · Retrieval Architectures · Memory Systems · Agentic Systems · Production AI Engineering · Distributed Systems

**Focus:** Designing reliable, cost-efficient, secure, observable, and context-aware LLM applications.

**Document Type:** Technical Engineering Reference

---

# 1. FOUNDATIONS

## 1.1 What Is Context Engineering?

Context Engineering is the discipline of designing, selecting, structuring, managing, validating, and delivering the information an LLM needs at each stage of execution.

It determines:

- **What:** Which information enters the model's context.
- **Source:** Where the information originates.
- **Selection:** Which information is relevant to the current task.
- **Prioritization:** How information is ranked and organized.
- **Timing:** When information is retrieved, updated, or removed.
- **Control:** Which information must be excluded.
- **Adaptation:** How context changes throughout execution.
- **Validation:** Whether assembled context is complete, reliable, authorized, and safe.
- **Budgeting:** How information is allocated within token, latency, and cost constraints.

### Primary Objective

Maximize answer quality, task completion, and system reliability while optimizing:

- Token consumption.
- Inference cost.
- Latency.
- Information relevance.
- Retrieval quality.
- Context utilization.
- Operational complexity.
- Security and reliability.

**Core Principle:**

Model intelligence alone does not determine application quality. The relevance, completeness, reliability, structure, and timing of supplied information significantly influence model outputs and decisions.

Context Engineering is broader than prompt engineering. It involves designing the complete information pipeline surrounding the model.

## 1.2 Context Engineering vs Prompt Engineering

| Dimension | Prompt Engineering | Context Engineering |
|---|---|---|
| Primary focus | Instructions and input wording | Complete information environment |
| Scope | Prompts and templates | Retrieval, memory, state, tools, instructions |
| Optimization | Instruction clarity | Relevance, completeness, cost, reliability |
| Information | Primarily supplied input | Multiple dynamic information sources |
| Lifecycle | Prompt creation and iteration | Retrieval, selection, updating, persistence |
| Evaluation | Prompt/output quality | Entire context pipeline and downstream outcomes |

Prompt engineering is one component of context engineering, not a replacement for it.

## 1.3 LLM Application Architecture

LLM applications can be organized into four broad architectural patterns.

| Architecture | Description | Best suited for |
|---|---|---|
| LLM Calls | Direct input-output interactions | Generation, classification, extraction |
| Workflows | Predetermined sequences of operations | Predictable, repeatable tasks |
| Agents | Models dynamically select tools and actions | Tasks requiring adaptive decisions |
| Agentic Systems | Coordinate agents, workflows, tools, memory, and execution state | Complex multi-stage objectives |

### Design Principles

1. Prefer deterministic workflows wherever possible.
2. Introduce autonomous decision-making only when the task requires it.
3. Use explicit state transitions to manage execution.
4. Separate model decisions from deterministic application logic.
5. Add validation, retries, and failure handling around external operations.
6. Make execution observable and recoverable.
7. Avoid unnecessary agent-to-agent communication.
8. Define termination conditions and execution budgets.

An agent is not automatically better than a workflow. Greater autonomy introduces additional complexity, cost, latency, and failure possibilities.

## 1.4 LLM Limitations

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
| Prompt Injection | Malicious content may attempt to manipulate model behavior |
| Context Dilution | Excessive irrelevant information can reduce context effectiveness |
| Model Limitations | Models may fail at reasoning, planning, or following complex instructions |
| Temporal Uncertainty | Information may become invalid as external conditions change |

**Engineering Implication:**

Production LLM systems must compensate through context selection, retrieval validation, structured outputs, evaluation, observability, authorization, and controlled execution.

---

# 2. THE SIX SOURCES OF CONTEXT

A production LLM application can assemble information from six major logical sources.

| Source | Purpose | Examples |
|---|---|---|
| Instructions | Define model behavior and constraints | System prompts, policies, task instructions |
| External Knowledge | Supply factual information | Documents, databases, APIs, knowledge bases |
| Tools | Enable external computation and actions | Python, web search, APIs, database queries |
| Memory | Retain useful information across interactions | User preferences, previous decisions |
| State | Track current execution and progress | Workflow step, retry count, pending tasks |
| User Prompts | Express immediate user intent | Questions, goals, constraints |

These are logical categories, not necessarily six separate services or databases.

## 2.1 Instructions

Instructions establish behavioral and operational boundaries.

They define:

- What the model should accomplish.
- Which rules it must follow.
- Which actions it can request.
- Which constraints it must respect.
- How outputs should be structured.

Examples:

- System instructions.
- Developer instructions.
- Task-specific instructions.
- Output schemas.
- Tool usage policies.
- Safety and authorization requirements.

**Engineering Considerations:**

- Define instruction precedence.
- Separate trusted instructions from untrusted retrieved content.
- Version prompts and instruction templates.
- Test conflicting instructions.
- Keep authorization decisions outside model-generated instructions.
- Avoid relying on prompts as the only security control.

## 2.2 External Knowledge

External knowledge supplies information that may not exist in pretrained knowledge or current conversation context.

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
- Provenance.

External knowledge is generally retrieved on demand rather than permanently inserting an entire knowledge base into the model's context.

## 2.3 Tools

Tools extend an LLM application's capabilities beyond text generation.

Examples:

- Python execution.
- Web search.
- Database queries.
- File operations.
- External API calls.
- Code execution.

### Tool Execution Lifecycle

1. Model determines whether a tool is required.
2. Model generates a structured tool request.
3. Application validates the tool name and arguments.
4. Application checks authorization and execution policies.
5. Tool executes within defined limits.
6. Application validates and processes the result.
7. Result enters the next model context.
8. Model continues execution or produces a final response.

**Engineering Considerations:**

- Authorization.
- Argument validation.
- Timeouts.
- Bounded retries.
- Rate limits.
- Idempotency.
- Side-effect management.
- Tool output validation.
- Audit logging.
- Execution isolation.

A model requesting an action does not mean that action is authorized.

## 2.4 Memory

Memory retains useful information across interactions or tasks.

Examples:

- User preferences.
- Previous decisions.
- Project information.
- Long-term facts.
- Previous task outcomes.

Memory should be selective rather than storing every interaction.

Important memory operations:

- Extraction.
- Qualification.
- Storage.
- Retrieval.
- Updating.
- Consolidation.
- Expiration.
- Deletion.

Memory retrieval should consider relevance, recency, reliability, permissions, and the current task.

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

State enables controlled execution, recovery, and continuation.

### Memory vs State

| Dimension | Memory | State |
|---|---|---|
| Purpose | Retain useful information | Track current execution |
| Lifetime | Across interactions or tasks | During an execution or workflow |
| Examples | Preferences, previous decisions | Current node, retry count |
| Update pattern | Selective updates | Frequent deterministic updates |
| Storage | Database, vector store | Checkpoint store, execution database |
| Primary owner | Memory management layer | Execution engine |

State should generally be managed by application logic rather than relying exclusively on the LLM to remember execution progress.

## 2.6 User Prompts

User prompts communicate immediate intent and requirements.

They may contain:

- Questions.
- Objectives.
- Constraints.
- Preferences.
- Supporting information.
- Expected output formats.
- Additional instructions.

The application should distinguish current user instructions from previously stored preferences.

A newer explicit user instruction can supersede an older preference, subject to higher-priority instructions and security constraints.

---

# 3. CONTEXT ASSEMBLY ARCHITECTURE

Context Assembly transforms information from different sources into a structured input that an LLM can use for a particular execution step.

It is one of the central responsibilities of a production context engineering system.

## 3.1 Context Assembly Pipeline

A typical pipeline consists of:

1. **Input Processing:** Understand the task and extract requirements.
2. **Source Identification:** Determine which information sources are needed.
3. **Identity and Authorization:** Establish which information the user or tenant can access.
4. **Retrieval:** Fetch relevant documents, memories, and external data.
5. **Filtering:** Remove irrelevant, duplicate, unauthorized, or unreliable information.
6. **Ranking:** Prioritize information based on relevance and reliability.
7. **Conflict Resolution:** Identify contradictory or outdated information.
8. **Compression:** Reduce unnecessary tokens while preserving meaning.
9. **Assembly:** Combine instructions, knowledge, memory, state, and user input.
10. **Budget Enforcement:** Check token allocation and reserve output capacity.
11. **Validation:** Check required fields, permissions, provenance, and conflicts.
12. **Delivery:** Supply the assembled context to the LLM.

The pipeline may be iterative. A tool result or model decision can trigger additional retrieval and context assembly.

## 3.2 Context Prioritization

Not every available piece of information deserves equal priority.

| Factor | Purpose |
|---|---|
| Relevance | Match with the current task |
| Authority | Trustworthiness of the source |
| Recency | Whether information is current |
| Reliability | Confidence in the information |
| Importance | Impact on the final decision |
| Permissions | Whether information is authorized |
| Provenance | Ability to trace information to its source |
| Specificity | How directly the information addresses the requirement |

**Engineering Principle:**

Relevance alone is insufficient. A highly relevant but outdated or unauthorized document should not automatically enter the context.

Authorization is a hard constraint, not merely another ranking weight.

## 3.3 Context Budget Management

LLMs operate within finite context windows. Context assembly must allocate tokens deliberately.

A conceptual input budget:

\[
T_{input} =
T_{instructions} +
T_{history} +
T_{retrieval} +
T_{memory} +
T_{state} +
T_{tools}
\]

The total input must fit within the model's effective context capacity after accounting for reserved output tokens and any model-specific overhead.

\[
T_{input} + T_{output} + T_{overhead}
\leq T_{capacity}
\]

Where:

- \(T_{instructions}\): Instruction tokens.
- \(T_{history}\): Relevant conversation history.
- \(T_{retrieval}\): Retrieved knowledge.
- \(T_{memory}\): Retrieved memory.
- \(T_{state}\): Execution state.
- \(T_{tools}\): Tool descriptions and results.
- \(T_{capacity}\): Available context capacity.

### Budget Management Strategies

- Dynamic allocation based on task complexity.
- Reserve output tokens before constructing input.
- Limit retrieval results using relevance thresholds.
- Remove redundant information.
- Compress long conversation history.
- Prioritize essential instructions and evidence.
- Track token usage at individual execution stages.
- Use model-specific tokenization rather than character-count estimates.

Context budgets should be dynamic rather than allocating fixed tokens to every source.

## 3.4 Context Structure

Context should be organized into clearly identifiable sections.

Example logical structure:

```text
SYSTEM INSTRUCTIONS
- Model behavior
- Safety constraints
- Output requirements

TASK CONTEXT
- Current objective
- User requirements
- Constraints

EXECUTION STATE
- Current step
- Previous results
- Pending actions

RETRIEVED KNOWLEDGE
- Documents
- Evidence
- Source identifiers
- Timestamps

MEMORY
- Relevant preferences
- Previous decisions

TOOL INFORMATION
- Available capabilities
- Validated tool results
```

The actual message structure depends on the model API and application architecture.

## 3.5 Context Provenance

Every important context item should be traceable to its origin.

Useful metadata:

```text
context_item_id
source_type
source_id
tenant_id
user_id
retrieved_at
source_updated_at
version
trust_level
access_scope
```

Provenance helps with:

- Debugging incorrect answers.
- Identifying outdated information.
- Auditing decisions.
- Reproducing execution behavior.
- Investigating data leakage.
- Evaluating source quality.

Do not confuse source provenance with factual correctness. A traceable source can still contain incorrect information.

---

# 4. RETRIEVAL ARCHITECTURES

Retrieval provides external information to an LLM when required knowledge is not already available in its context.

## 4.1 Retrieval-Augmented Generation (RAG)

RAG combines information retrieval with LLM generation.

### Typical RAG Pipeline

1. Document ingestion.
2. Text extraction and cleaning.
3. Document normalization.
4. Document chunking.
5. Metadata extraction.
6. Embedding generation.
7. Vector database storage.
8. User query processing.
9. Retrieval.
10. Candidate ranking.
11. Context construction.
12. LLM generation.
13. Response validation.
14. Citation or source attribution.

### Document Ingestion Considerations

- Preserve document identifiers.
- Track document versions.
- Extract useful metadata.
- Handle malformed files.
- Detect duplicate documents.
- Support incremental updates.
- Maintain document-to-chunk relationships.
- Define deletion and re-indexing policies.

## 4.2 Retrieval Strategies

| Strategy | Description | Use Case |
|---|---|---|
| Dense Retrieval | Embedding-based semantic search | Conceptual queries |
| Sparse Retrieval | Keyword-based search such as BM25 | Exact terms and identifiers |
| Hybrid Retrieval | Combines dense and sparse retrieval | General-purpose retrieval |
| Metadata Filtering | Restricts results using structured attributes | Tenant, date, document type |
| Reranking | Reorders candidates using a relevance model | Improving top-K relevance |
| Query Rewriting | Transforms the original query | Ambiguous queries |
| Multi-Query Retrieval | Generates multiple search queries | Broader information coverage |

## 4.3 Advanced Retrieval Techniques

**Query Decomposition:** Break complex questions into smaller retrieval tasks.

**Contextual Compression:** Extract relevant portions of retrieved documents while preserving necessary meaning.

**Parent-Child Retrieval:** Retrieve smaller chunks for precision while retaining access to their larger document context.

**Recursive Retrieval:** Perform additional retrieval based on information discovered in earlier stages.

**Corrective Retrieval:** Detect insufficient or unreliable results and trigger alternative retrieval strategies.

**Hybrid Fusion:** Combine dense and sparse search rankings using techniques such as Reciprocal Rank Fusion (RRF).

**Self-Query Retrieval:** Translate natural-language constraints into structured metadata filters, with application-side validation.

**Graph-Based Retrieval:** Use relationships between entities to retrieve connected information.

**Temporal Retrieval:** Account for time-sensitive facts, document versions, and validity periods.

## 4.4 Hybrid Search and Reciprocal Rank Fusion

Dense and sparse retrieval produce different ranking signals. Their raw scores may not be directly comparable.

RRF combines rankings using:

\[
RRF(d) = \sum_{r \in R}
\frac{1}{k + rank_r(d)}
\]

Where:

- \(d\): Retrieved document.
- \(R\): Set of ranking systems.
- \(rank_r(d)\): Position of document \(d\) in ranking system \(r\).
- \(k\): Ranking constant.

RRF is useful because it combines rankings without requiring directly comparable raw retrieval scores.

## 4.5 Retrieval Failure Handling

A production retrieval pipeline should define behavior for:

- No relevant documents found.
- Low-confidence retrieval.
- Contradictory documents.
- Outdated information.
- Unauthorized results.
- Retrieval service timeouts.
- Embedding service failures.
- Database unavailability.
- Malformed retrieval responses.

Possible responses:

- Retry with bounded limits.
- Rewrite or decompose the query.
- Fall back to another retrieval strategy.
- Ask the user for clarification.
- Return an explicit insufficient-evidence response.
- Escalate to human review where appropriate.

Never silently replace missing evidence with fabricated information.

## 4.6 RAG Evaluation

Evaluate retrieval separately from generation.

Important metrics:

| Metric | Purpose |
|---|---|
| Precision@K | Fraction of retrieved results that are relevant |
| Recall@K | Fraction of relevant results retrieved within top K |
| MRR | Position of the first relevant result |
| NDCG@K | Ranking quality with graded relevance |
| Hit Rate@K | Whether at least one relevant result appears |
| Context Precision | Relevance of information supplied to generation |
| Context Recall | Coverage of expected supporting information |

A model cannot reliably answer using information that the retrieval system failed to provide.

---

# 5. MEMORY ARCHITECTURES

Memory systems allow applications to retain and reuse information beyond an individual model call.

## 5.1 Memory Types

| Memory Type | Description | Examples |
|---|---|---|
| Working Memory | Information required for the current task | Current context, intermediate results |
| Short-Term Memory | Information retained within a limited interaction | Recent conversation |
| Long-Term Memory | Information retained across sessions | Stable user preferences |
| Episodic Memory | Records of previous experiences | Previous tasks and outcomes |
| Semantic Memory | Generalized facts and knowledge | Stable facts and concepts |
| Procedural Memory | Knowledge about performing tasks | Reusable workflows |

These categories are conceptual and can overlap. They do not necessarily require separate databases.

## 5.2 Memory Lifecycle

A production memory system should implement:

1. **Extraction:** Identify potentially useful information.
2. **Qualification:** Determine whether it is worth retaining.
3. **Storage:** Save with appropriate metadata.
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
- Temporal validity.

A conceptual ranking function:

\[
Score(m,q) =
w_rR(m,q) + w_iI(m) + w_tT(m) + w_cC(m)
\]

Where:

- \(R(m,q)\): Relevance to the current query.
- \(I(m)\): Importance of the memory.
- \(T(m)\): Temporal relevance.
- \(C(m)\): Confidence or reliability.
- \(w_r,w_i,w_t,w_c\): Tunable weights.

This is an illustrative ranking model, not a universal scoring formula.

Permissions and deletion status must be enforced before ranking.

## 5.4 Memory Quality and Conflict Resolution

A production memory system must handle:

- Duplicate memories.
- Contradictory information.
- Outdated preferences.
- Incorrect extracted facts.
- Temporary information stored as permanent.
- Sensitive information.
- User-requested deletion.
- Incorrect memory associations.

Useful metadata includes:

```text
memory_id
user_id
tenant_id
memory_type
content
source
created_at
updated_at
last_accessed_at
confidence
importance
expiration_at
version
```

### Conflict Resolution Principles

1. Prefer explicit corrections over older inferred information.
2. Consider source reliability.
3. Track when information was last verified.
4. Preserve uncertainty instead of converting guesses into facts.
5. Apply expiration to temporary information.
6. Do not automatically overwrite reliable facts with ambiguous statements.
7. Support deletion and correction workflows.

**Engineering Principle:**

Memory quality depends on what the system stores, what it retrieves, when it retrieves it, and whether it correctly updates old information.

---

# 6. CONTEXT MANAGEMENT IN AGENTIC SYSTEMS

Context requirements change throughout agent execution.

An agent may need different information when planning, selecting tools, executing operations, validating results, and producing its final response.

## 6.1 Agent Execution Cycle

1. Receive the user request.
2. Assemble initial context.
3. Analyze the task.
4. Select an action or tool.
5. Validate the requested action.
6. Execute the action.
7. Process the tool result.
8. Update execution state.
9. Retrieve additional context if required.
10. Evaluate progress and termination conditions.
11. Continue or terminate execution.
12. Produce the final response.

## 6.2 Context Management Strategies

| Strategy | Purpose |
|---|---|
| Context Pruning | Remove irrelevant information |
| Context Compression | Reduce token usage while retaining meaning |
| Context Summarization | Represent lengthy history in shorter form |
| Just-in-Time Retrieval | Retrieve information only when needed |
| Context Isolation | Separate information between users, tenants, or tasks |
| Context Refreshing | Replace stale information with updated information |
| Context Checkpointing | Preserve information required to resume execution |
| Context Caching | Reuse suitable context or computation |
| Context Versioning | Track changes in prompts, sources, and assembly policies |

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
- Execution deadlines.
- Cancellation handling.
- Concurrency control.

### Checkpointing

A checkpoint should preserve the information needed to resume execution safely.

It may include:

- Workflow identifier.
- Execution identifier.
- Current execution node.
- Completed steps.
- Pending operations.
- Relevant intermediate results.
- Retry metadata.
- Approval status.
- State version.
- Checkpoint timestamp.

A checkpoint should not automatically preserve every token from the previous model context.

### Checkpoint Consistency

A production checkpointing system must consider:

- Atomic persistence.
- State versioning.
- Concurrent updates.
- Partial writes.
- Corrupted checkpoints.
- Schema migrations.
- Recovery from storage failure.

A checkpoint should represent a consistent execution state, not an arbitrary snapshot of partially completed operations.

## 6.4 Failure Recovery and Side Effects

Consider a workflow that performs an external payment.

A crash may happen after the payment succeeds but before the application records success.

On retry, the application may accidentally execute the payment again.

The system must account for this boundary using:

- Idempotency keys.
- Durable operation records.
- External operation identifiers.
- Reconciliation.
- Safe retry policies.
- Transactional outbox patterns where applicable.
- Explicit operation status transitions.

**Important:**

Checkpointing alone does not guarantee exactly-once execution of external side effects.

A practical architecture generally aims for at-least-once delivery with idempotent processing and reconciliation.

## 6.5 Human-in-the-Loop (HITL)

Some operations require explicit human approval.

Examples:

- Financial transactions.
- Destructive operations.
- High-impact external actions.
- Sensitive data access.
- Irreversible workflow transitions.

A typical HITL lifecycle:

1. Agent proposes an action.
2. Application validates the proposal.
3. Workflow enters a pending-approval state.
4. Authorized human reviews the action.
5. Approval or rejection is recorded.
6. Workflow resumes or terminates.

Approval should be bound to the specific action, relevant parameters, execution identity, and authorization context.

---

# 7. CONTEXT CACHING AND OPTIMIZATION

Context management also involves reducing repeated computation, latency, and inference costs.

## 7.1 Caching Strategies

| Strategy | Purpose |
|---|---|
| Retrieval Cache | Reuse suitable retrieval results |
| Embedding Cache | Avoid recomputing embeddings for identical content |
| Query Cache | Reuse results for suitable repeated queries |
| Semantic Cache | Reuse responses for sufficiently equivalent queries |
| Prompt Prefix Cache | Reuse supported repeated prompt prefixes |
| Application Cache | Cache frequently accessed application data |

## 7.2 Cache Invalidation

Caching introduces consistency problems.

A production system should define:

- Cache expiration.
- Version-based invalidation.
- Document update handling.
- User and tenant isolation.
- Permission-aware cache keys.
- Stale-result detection.
- Invalidation after deletion.
- Negative-result caching policies.

Never allow a shared cache to expose one tenant's retrieved context to another tenant.

### Cache Key Design

A cache key may incorporate:

```text
tenant_id
user_id_or_access_scope
query_hash
embedding_model_version
retrieval_configuration_version
document_index_version
permission_scope_version
```

The exact key depends on the cached object and its security requirements.

## 7.3 Semantic Cache Risks

Semantic caching can incorrectly treat different queries as equivalent.

For example:

- "Show my account balance."
- "Show my previous account balance."

These queries may be semantically similar but require different results.

Therefore:

- Use task-aware similarity thresholds.
- Apply authorization before cache reuse.
- Include relevant user and tenant scope.
- Avoid caching sensitive or rapidly changing results without suitable controls.
- Validate whether the cached response remains applicable.

## 7.4 Cost Optimization

Measure:

- Tokens per execution.
- Model calls per task.
- Retrieval cost.
- Embedding cost.
- Reranking cost.
- Tool execution cost.
- Cache hit rates.
- Total cost per successful task.

Optimization techniques:

- Retrieve only relevant information.
- Use metadata filtering early.
- Avoid unnecessary model calls.
- Use smaller models for suitable tasks.
- Compress context selectively.
- Cache reusable computation.
- Avoid unnecessary agent iterations.
- Set execution budgets and stopping conditions.

---

# 8. CONTEXT EVALUATION AND OBSERVABILITY

Context engineering must be measurable.

A context pipeline that appears effective in a demonstration may fail under real-world workloads.

## 8.1 Evaluation Dimensions

| Dimension | What to Measure |
|---|---|
| Retrieval Quality | Recall@K, Precision@K, MRR, NDCG |
| Context Relevance | Relevance of supplied information |
| Context Completeness | Whether essential information was included |
| Context Freshness | Whether supplied information is current |
| Groundedness | Whether outputs are supported by evidence |
| Task Success | Whether the application completed the intended task |
| Token Efficiency | Tokens consumed per successful task |
| Latency | Retrieval, assembly, and generation latency |
| Cost | Cost per execution and successful task |
| Reliability | Failure rate, timeout rate, recovery success |
| Security | Unauthorized access and data leakage attempts |

## 8.2 Context-Level Testing

Test the context pipeline independently from final LLM output.

Examples:

- Did retrieval return the expected documents?
- Were unauthorized documents excluded?
- Was relevant memory retrieved?
- Was stale information filtered?
- Were conflicting facts detected?
- Did compression preserve important details?
- Did token-budget enforcement work?
- Did tenant isolation prevent cross-user leakage?
- Did the system correctly handle empty retrieval results?
- Were source identifiers preserved?
- Did context assembly use the correct prompt version?

## 8.3 Retrieval Evaluation

Important retrieval metrics:

**Precision@K:** Fraction of retrieved results that are relevant.

**Recall@K:** Fraction of all relevant results retrieved within the top K.

**MRR:** Measures how highly the first relevant result appears.

**NDCG:** Evaluates ranking quality while accounting for relevance and position.

Retrieval evaluations should use representative queries and labeled relevant documents.

## 8.4 Observability

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
- Cache hit rates.
- Checkpoint creation and recovery events.
- Tenant-scoped execution identifiers.

### Distributed Tracing

A useful trace hierarchy:

```text
API Request
  └── Workflow Execution
       ├── Context Assembly
       │    ├── Memory Retrieval
       │    ├── Document Retrieval
       │    └── Context Validation
       ├── LLM Inference
       ├── Tool Execution
       ├── State Persistence
       └── Final Response
```

Use trace IDs and span IDs to connect operations across services.

Sensitive context should not be indiscriminately logged.

Apply redaction, access controls, and retention policies.

## 8.5 Evaluation Harness

A practical evaluation harness consists of three major components.

| Component | Responsibility |
|---|---|
| Evaluator | Measures context and output quality |
| Runner | Executes test cases against the system |
| Reporter | Aggregates results and identifies regressions |

Recommended testing layers:

1. Retrieval component evaluations.
2. Memory retrieval and update evaluations.
3. Context assembly tests.
4. Tool execution tests.
5. State and checkpoint tests.
6. End-to-end task evaluations.
7. Regression evaluations.
8. Failure injection and recovery tests.
9. Security and isolation tests.

Evaluation datasets should include normal, ambiguous, adversarial, malformed, and failure scenarios.

### Regression Management

- Maintain versioned evaluation datasets.
- Compare candidate and baseline versions.
- Define acceptable regression thresholds.
- Track failures by category.
- Preserve failed test cases.
- Require review for significant quality regressions.

LLM-as-a-judge can assist evaluation, but it should not be treated as unquestionable ground truth. Use deterministic checks, human-reviewed samples, and calibrated evaluators where appropriate.

---

# 9. SECURITY AND TRUST BOUNDARIES

Security must be enforced before information reaches the model and before model-requested actions are executed.

## 9.1 Context Security

Important controls:

- Tenant and user isolation.
- Permission-aware retrieval.
- Authorization before database access.
- Memory access restrictions.
- Sensitive information redaction.
- Data retention and deletion policies.
- Secure logging.
- Credential isolation.
- Least-privilege tool execution.

## 9.2 Prompt Injection

Retrieved documents, websites, and tool outputs may contain malicious instructions.

For example, a retrieved document might attempt to instruct the model to reveal secrets or ignore previous instructions.

Defenses include:

- Treat retrieved content as untrusted data.
- Clearly separate instructions from external content.
- Validate tool calls independently.
- Restrict available tools.
- Enforce authorization in application code.
- Avoid exposing secrets to model context.
- Test indirect prompt injection scenarios.
- Validate outputs before performing sensitive actions.

**Important:** Prompt instructions alone are not a security boundary. Critical permissions must be enforced by deterministic application logic.

## 9.3 Multi-Tenant Context Isolation

For multi-tenant systems:

- Every tenant-owned record should have tenant ownership.
- Every query must enforce tenant scope.
- Retrieval filters must include authorization constraints.
- Cache keys must include appropriate tenant identity.
- Memory must be isolated.
- Tool execution must use authorized credentials.
- Cross-tenant access must be tested explicitly.
- Background workers must preserve tenant context.
- Logs and observability data must respect access boundaries.

Where possible, enforce isolation through database constraints, row-level security, scoped repositories, and authorization middleware.

A model should never be responsible for enforcing tenant isolation by itself.

## 9.4 Data Lifecycle and Privacy

Production systems should define:

- Data classification.
- Data minimization.
- Encryption in transit and at rest.
- Retention periods.
- Deletion propagation.
- Backup retention.
- Audit trails.
- Access reviews.
- Sensitive information handling.

Deleting a memory from one application table may not automatically delete associated embeddings, caches, logs, backups, or derived records.

Deletion workflows must account for the complete data lifecycle.

---

# 10. PRODUCTION ENGINEERING PRINCIPLES

Context Engineering is not only an LLM concern. It intersects with backend engineering, distributed systems, security, and reliability.

## 10.1 Reliability

- Use bounded retries.
- Validate structured outputs.
- Implement timeouts.
- Persist critical execution state.
- Use idempotency for side-effecting operations.
- Implement checkpoint recovery.
- Fail safely when required context is unavailable.
- Define fallback behavior for dependency failures.
- Implement cancellation and deadline handling.
- Monitor dependency health.
- Reconcile uncertain external operations.

## 10.2 Performance

- Retrieve only relevant information.
- Use metadata filtering before expensive retrieval.
- Cache suitable results.
- Compress context selectively.
- Monitor token consumption.
- Avoid unnecessary model calls.
- Optimize expensive retrieval stages independently.
- Parallelize independent operations when safe.
- Set concurrency limits.
- Monitor tail latency, not only average latency.

## 10.3 Distributed Systems Considerations

A multi-service AI application may involve:

- API gateway.
- Authentication service.
- Workflow execution service.
- Context assembly service.
- Retrieval service.
- Memory service.
- Model provider.
- Tool execution workers.
- State persistence.
- Evaluation service.
- Observability infrastructure.

Important distributed systems concepts:

| Concept | Application |
|---|---|
| Idempotency | Prevent duplicate side effects |
| At-Least-Once Delivery | Handle message redelivery safely |
| Concurrency Control | Prevent conflicting execution updates |
| Durable Queues | Preserve work across service restarts |
| Dead-Letter Queues | Isolate repeatedly failing tasks |
| Circuit Breakers | Prevent cascading dependency failures |
| Timeouts | Bound resource consumption |
| Backpressure | Control overloaded services |
| Transactional Outbox | Reliably coordinate database writes and event publication |
| Reconciliation | Resolve uncertain external operation outcomes |

## 10.4 Architectural Separation

Keep these responsibilities clearly separated:

- Context retrieval.
- Memory management.
- Context assembly.
- Model inference.
- Tool execution.
- State persistence.
- Evaluation.
- Observability.
- Authorization.
- Cost management.

This separation makes individual components easier to test, optimize, replace, and debug.

## 10.5 Practical Production Architecture

A conceptual architecture:

```text
                  USER
                    |
                    v
              API GATEWAY
                    |
                    v
          AUTHENTICATION / RBAC
                    |
                    v
          WORKFLOW ORCHESTRATOR
                    |
          +---------+----------+
          |                    |
          v                    v
   CONTEXT ASSEMBLY       EXECUTION STATE
          |                    |
    +-----+------+             |
    |     |      |             |
    v     v      v             |
  MEMORY RAG  EXTERNAL         |
 SERVICE     KNOWLEDGE         |
    |     |      |             |
    +-----+------+             |
          |                    |
          v                    |
    CONTEXT VALIDATION         |
          |                    |
          v                    |
      LLM INFERENCE             |
          |                    |
          v                    |
     TOOL EXECUTION             |
          |                    |
          v                    |
    RESULT VALIDATION          |
          |                    |
          +---------+----------+
                    |
                    v
          STATE / MEMORY UPDATE
                    |
                    v
              FINAL OUTPUT

    OBSERVABILITY + EVALUATION
    ACROSS THE ENTIRE SYSTEM
```

This is a logical architecture, not a mandatory microservices deployment. Small applications can implement these responsibilities as modules inside a single service.

---

# 11. DESIGN DECISIONS AND ENGINEERING TRADE-OFFS

Production engineering requires choosing between competing approaches.

| Decision | Option A | Option B | Trade-off |
|---|---|---|---|
| Retrieval | Dense | Hybrid | Semantic simplicity vs broader retrieval coverage |
| Memory | Relational | Vector-based | Structured consistency vs semantic retrieval |
| Execution | Workflow | Autonomous agent | Predictability vs flexibility |
| Context | Full history | Selective history | Completeness vs token efficiency |
| Deployment | Monolith | Microservices | Operational simplicity vs independent scaling |
| State | In-memory | Persistent checkpoints | Simplicity vs recovery |
| Caching | Exact | Semantic | Precision vs reuse opportunities |
| Evaluation | Deterministic | Model-based judge | Repeatability vs semantic flexibility |

### Decision Principles

1. Start with the simplest architecture that satisfies requirements.
2. Measure before introducing complexity.
3. Define explicit failure behavior.
4. Consider operational cost, not only implementation cost.
5. Prefer reversible architectural decisions where possible.
6. Document important decisions using Architecture Decision Records (ADRs).
7. Revisit decisions when production evidence contradicts assumptions.

---

# 12. PRACTICAL ENGINEERING CHECKLIST

Use this checklist when designing or reviewing a production LLM application.

## Context Assembly
- [ ] Context sources are clearly defined.
- [ ] Retrieval is task-dependent.
- [ ] Context budgets are enforced.
- [ ] Provenance is preserved.
- [ ] Stale and contradictory information is handled.
- [ ] Unauthorized information is excluded.

## Retrieval
- [ ] Ingestion pipeline is versioned.
- [ ] Chunking strategy is evaluated.
- [ ] Metadata filters are enforced.
- [ ] Hybrid retrieval is evaluated where appropriate.
- [ ] Reranking is measured against a baseline.
- [ ] Empty and failed retrieval behavior is defined.

## Memory
- [ ] Memory extraction is selective.
- [ ] Memory retrieval is permission-aware.
- [ ] Updates and conflicts are handled.
- [ ] Expiration policies exist.
- [ ] Deletion propagates to derived data.

## Agent Execution
- [ ] Execution state is persisted.
- [ ] Checkpoint recovery is tested.
- [ ] Side effects are idempotent.
- [ ] Retry policies are bounded.
- [ ] Human approval is supported where required.
- [ ] Cancellation and termination conditions exist.

## Security
- [ ] Authentication and authorization are enforced.
- [ ] Tenant isolation is tested.
- [ ] Prompt injection scenarios are tested.
- [ ] Tool permissions are restricted.
- [ ] Secrets are not exposed unnecessarily.
- [ ] Sensitive logging is controlled.

## Evaluation and Observability
- [ ] Retrieval has a labeled evaluation dataset.
- [ ] Context assembly has independent tests.
- [ ] Memory retrieval has regression tests.
- [ ] End-to-end task success is measured.
- [ ] Cost and latency are tracked.
- [ ] Traces connect execution stages.
- [ ] Failure injection is part of testing.

---

# 13. FINAL ENGINEERING MODEL

A useful mental model for production Context Engineering is:

**Context Sources → Identity and Authorization → Retrieval → Filtering → Ranking → Context Assembly → Validation → LLM Execution → Result Validation → State and Memory Updates → Evaluation**

This is a logical architecture, not a mandatory linear execution sequence.

Production systems may repeat retrieval, assembly, and execution as workflows progress.

## Core Takeaways

1. Context Engineering is broader than prompt engineering.
2. Context must be selected, not simply accumulated.
3. Memory and execution state are different architectural responsibilities.
4. Retrieval quality directly influences downstream generation quality.
5. Context budgets must account for cost, latency, and model limits.
6. Agents require dynamic context management throughout execution.
7. Context pipelines need independent evaluation and observability.
8. Security and tenant isolation must be enforced before information reaches the model.
9. Deterministic application logic should control critical operations.
10. Checkpointing does not automatically guarantee safe recovery.
11. Caching requires explicit invalidation and isolation policies.
12. Provenance helps trace information but does not prove correctness.
13. Distributed execution requires idempotency, concurrency control, and recovery.
14. Evaluation should cover both individual components and end-to-end behavior.
15. Architectural complexity should be justified by measurable requirements.

## FINAL PRINCIPLE

A reliable LLM application does not simply provide more context.

It provides the right information, from the right source, at the right time, in the right structure, with the right permissions, validation, and execution controls.

**The objective is not maximum context utilization. The objective is maximum useful context with minimum unnecessary complexity, cost, and risk.**