# CONTEXT ENGINEERING — PRODUCTION TECHNICAL REFERENCE

**Topics:** LLM System Design · Context Engineering · Retrieval Architectures · Memory Systems · Agentic Systems · Production AI Engineering · Distributed Systems

**Focus:** Designing reliable, cost-efficient, secure, observable, and context-aware LLM applications.

**Document Type:** Technical Engineering Reference

---

# 0. EXECUTIVE MODEL

Context Engineering is the discipline of controlling the information available to an LLM throughout an application's lifecycle.

The objective is not to maximize the amount of context. It is to provide the minimum sufficient, authorized, relevant, reliable, and timely context required for the current decision.

### Core Context Pipeline

```text
User Request
    |
    v
Identity + Authorization
    |
    v
Task / Context Requirements
    |
    v
Retrieve from Approved Sources
    |
    v
Filter -> Rank -> Resolve Conflicts -> Compress
    |
    v
Context Assembly + Token Budget
    |
    v
Context Validation
    |
    v
LLM / Workflow Execution
    |
    v
Output / Tool Request Validation
    |
    v
Authorized Tool Execution
    |
    v
State and Memory Updates
    |
    v
Evaluation + Observability
```

This is a logical model, not a mandatory linear execution sequence. Retrieval, context assembly, inference, and validation may repeat throughout an agent's execution.

### Fundamental Distinctions

- **Prompt Engineering:** Designs instructions and input wording.
- **Context Engineering:** Manages the complete information environment supplied to the model.
- **Memory:** Retains selected information for future use.
- **Execution State:** Records the current progress and status of a task.
- **Retrieval:** Fetches external information when required.
- **Authorization:** Determines which information and actions are permitted.

Authorization must be enforced by application logic, not delegated to the LLM.

---

# 1. FOUNDATIONS

## 1.1 What Is Context Engineering?

Context Engineering is the discipline of designing, selecting, structuring, managing, validating, and delivering the information an LLM needs at each stage of execution.

It determines:

| Dimension | Engineering Question |
|---|---|
| What | Which information is required? |
| Source | Where does the information originate? |
| Selection | Is it relevant to the current task? |
| Prioritization | Which information matters most? |
| Timing | When should information be retrieved or refreshed? |
| Exclusion | What information must not enter the context? |
| Adaptation | How should context change during execution? |
| Validation | Is the information authorized, complete, and reliable? |
| Budgeting | Does it fit within token, latency, and cost constraints? |
| Provenance | Can important information be traced to its source? |

### Primary Objective

Maximize answer quality and task completion while optimizing:

- Token consumption.
- Inference cost.
- Latency.
- Retrieval quality.
- Information relevance.
- Context freshness.
- Operational complexity.
- Security and reliability.

**Core Principle:**

Model intelligence alone does not determine application quality. The relevance, completeness, reliability, structure, and timing of supplied information significantly influence model outputs and decisions.

## 1.2 Context Engineering vs Prompt Engineering

| Dimension | Prompt Engineering | Context Engineering |
|---|---|---|
| Primary focus | Instructions and input wording | Complete information environment |
| Scope | Prompts and templates | Retrieval, memory, state, tools, instructions |
| Optimization | Instruction clarity | Relevance, completeness, cost, reliability |
| Information | Primarily supplied input | Multiple dynamic information sources |
| Lifecycle | Prompt creation and iteration | Retrieval, selection, updating, persistence |
| Evaluation | Prompt/output quality | Entire context pipeline and downstream outcomes |

Prompt Engineering is one component of Context Engineering.

## 1.3 LLM Application Architecture

LLM applications can be organized into four broad architectural patterns.

| Architecture | Description | Suitable For |
|---|---|---|
| LLM Calls | Direct input-output interactions | Generation, classification, extraction |
| Workflows | Predetermined sequences of operations | Predictable, repeatable tasks |
| Agents | Models dynamically select tools and actions | Adaptive decision-making |
| Agentic Systems | Coordinate agents, workflows, tools, memory, and execution state | Complex multi-stage objectives |

### Design Principles

1. Prefer deterministic workflows wherever possible.
2. Introduce autonomy only when the task requires it.
3. Use explicit state transitions.
4. Separate model decisions from deterministic application logic.
5. Add validation, retries, and failure handling around external operations.
6. Make execution observable and recoverable.
7. Avoid unnecessary agent-to-agent communication.
8. Define termination conditions and execution budgets.

An agent is not automatically better than a workflow. Greater autonomy introduces additional complexity, cost, latency, and failure possibilities.

## 1.4 LLM Limitations

| Limitation | Description | Engineering Response |
|---|---|---|
| Hallucinations | Plausible but incorrect information | Grounding and validation |
| Stale Knowledge | Pretrained knowledge may be outdated | External retrieval |
| Context Limits | Finite context windows | Token budgeting |
| Lost-in-the-Middle | Position-dependent attention | Context ordering and testing |
| Information Integrity | Retrieved information may be unreliable | Source validation |
| Non-determinism | Outputs may vary | Evaluation and validation |
| Tool Failures | APIs and external systems may fail | Timeouts and bounded retries |
| Instruction Conflicts | Instructions may contradict | Instruction hierarchy |
| Prompt Injection | Malicious content manipulates behavior | Trust boundaries and authorization |
| Context Dilution | Irrelevant information reduces effectiveness | Context pruning |
| Model Limitations | Reasoning and planning failures | Decomposition and deterministic checks |
| Temporal Uncertainty | Information becomes outdated | Timestamps and refresh policies |

**Engineering Implication:**

Production LLM systems must compensate through retrieval validation, structured outputs, evaluation, observability, authorization, and controlled execution.

---

# 2. THE SIX SOURCES OF CONTEXT

A production LLM application can assemble information from six major logical sources.

| Source | Purpose | Examples |
|---|---|---|
| Instructions | Define model behavior and constraints | System prompts, policies, task instructions |
| External Knowledge | Supply factual information | Documents, databases, APIs |
| Tools | Enable computation and external actions | Python, search, database queries |
| Memory | Retain useful information | Preferences, previous decisions |
| Execution State | Track current execution | Workflow step, retry count |
| User Prompts | Express immediate intent | Questions, goals, constraints |

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
- Tool-use policies.
- Safety requirements.

### Engineering Considerations

- Define instruction precedence.
- Separate trusted instructions from untrusted retrieved content.
- Version prompts and instruction templates.
- Test conflicting instructions.
- Keep authorization decisions outside model-generated instructions.
- Never rely on prompts as the only security control.

## 2.2 External Knowledge

External knowledge supplies information that may not exist in pretrained knowledge or current conversation context.

Common sources:

- Vector databases.
- Relational databases.
- Document repositories.
- Search engines.
- External APIs.
- Knowledge graphs.

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

### Engineering Considerations

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

## 2.5 Execution State

Execution state describes the current condition and progress of an execution.

Examples:

- Current workflow step.
- Execution status.
- Retry count.
- Pending actions.
- Tool execution results.
- Checkpoint information.
- Human approval status.

State enables controlled execution, recovery, and continuation.

### Memory vs Execution State

| Dimension | Memory | Execution State |
|---|---|---|
| Purpose | Retain useful information | Track current execution |
| Lifetime | Across interactions or tasks | During an execution or workflow |
| Examples | Preferences, previous decisions | Current node, retry count |
| Update Pattern | Selective updates | Frequent deterministic updates |
| Storage | Database, vector store | Checkpoint store, execution database |
| Primary Owner | Memory management layer | Execution engine |

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

It is one of the central responsibilities of a production Context Engineering system.

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

| Factor | Purpose |
|---|---|
| Relevance | Match with the current task |
| Authority | Trustworthiness of the source |
| Recency | Whether information is current |
| Reliability | Confidence in the information |
| Importance | Impact on the final decision |
| Permissions | Whether information is authorized |
| Provenance | Ability to trace information to its source |
| Specificity | How directly information addresses the requirement |

**Engineering Principle:**

Relevance alone is insufficient. A highly relevant but outdated or unauthorized document should not automatically enter the context.

Authorization is a hard constraint, not merely another ranking weight.

## 3.3 Context Contracts

For important workflow steps, define an explicit context contract.

Example:

```text
step_name
required_context_fields
optional_context_fields
authorized_sources
freshness_requirements
maximum_token_budget
required_provenance
conflict_policy
missing_context_behavior
validation_rules
```

A context contract makes context requirements testable.

For example, a financial action step may require:

- Verified user identity.
- Current account information.
- Explicit authorization.
- Valid transaction parameters.
- Human approval.

Missing required information should stop the action rather than be guessed.

## 3.4 Context Budget Management

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

The total input must fit within the model's effective context capacity after accounting for reserved output tokens and model-specific overhead.

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

## 3.5 Context Structure

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

Labels improve organization but do not independently create security boundaries.

## 3.6 Context Provenance and Freshness

Every important context item should be traceable to its origin.

Useful metadata:

```text
context_item_id
source_type
source_id
tenant_id
user_id
source_version
access_scope
retrieved_at
source_updated_at
valid_from
valid_until
```

Provenance helps with:

- Debugging incorrect answers.
- Identifying outdated information.
- Auditing decisions.
- Reproducing execution behavior.
- Investigating data leakage.
- Evaluating source quality.

Do not confuse source provenance with factual correctness. A traceable source can still contain incorrect information.

For time-sensitive information, distinguish the source's update time from the time the application retrieved it.

---

# 4. RETRIEVAL ARCHITECTURES

Retrieval provides external information to an LLM when required knowledge is not already available in its context.

## 4.1 Retrieval-Augmented Generation (RAG)

RAG combines information retrieval with LLM generation.

### Typical RAG Pipeline

```text
DOCUMENT INGESTION
        |
        v
Extraction
        |
        v
Normalization
        |
        v
Chunking
        |
        v
Metadata Enrichment
        |
        v
Embedding Generation
        |
        v
Vector / Search Index Storage
        |
        v
QUERY PROCESSING
        |
        v
Access-Controlled Retrieval
        |
        v
Ranking / Fusion / Reranking
        |
        v
Context Construction
        |
        v
LLM Generation
        |
        v
Response Validation
        |
        v
Citation / Source Attribution
```

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
| Parent-Child Retrieval | Retrieves smaller chunks with broader context | Precision with surrounding meaning |
| Query Decomposition | Splits complex queries | Multi-part questions |
| Graph Retrieval | Uses entity relationships | Relationship-heavy queries |
| Temporal Retrieval | Accounts for time and validity | Time-sensitive information |

## 4.3 Advanced Retrieval Techniques

**Contextual Compression:** Extract relevant portions of retrieved documents while preserving necessary meaning and provenance.

**Recursive Retrieval:** Perform additional retrieval based on information discovered in earlier stages.

**Corrective Retrieval:** Detect insufficient or unreliable results and trigger alternative retrieval strategies.

**Hybrid Fusion:** Combine dense and sparse search rankings using techniques such as Reciprocal Rank Fusion (RRF).

**Self-Query Retrieval:** Translate natural-language constraints into structured metadata filters, with application-side validation.

**Graph-Based Retrieval:** Use relationships between entities to retrieve connected information.

**Temporal Retrieval:** Account for time-sensitive facts, document versions, and validity periods.

**Deduplication:** Prevent repeated passages from consuming context unnecessarily.

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
| Long-Term Memory | Information retained across sessions | Stable preferences |
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
last_verified_at
confidence
importance
expiration_at
retention_policy
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

