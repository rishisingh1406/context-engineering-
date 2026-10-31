# CONTEXT ENGINEERING — QUICK REFERENCE

A practical reference covering context engineering, retrieval architectures, and production LLM system design.

---

## 1. DEFINITION

**Context Engineering** is the discipline of selecting, structuring, managing, and delivering the right information to an LLM at each stage of execution.

**Primary Goal:**

Maximize answer quality and system reliability while optimizing:

- Token consumption
- Latency
- Inference cost
- Operational complexity
- Information relevance

**Core Idea:** The quality of an LLM system depends not only on the model but also on the quality of the context supplied to it.

---

## 2. LLM SYSTEM ARCHITECTURE

LLM-based systems can be organized into four broad categories.

### 1. LLM Calls
- Direct input-output interactions.
- The application controls the request and processes the response.
- Suitable for straightforward tasks.

### 2. Workflows
- Execute predefined sequences of operations.
- Follow predetermined control flow.
- Suitable for predictable and repeatable tasks.

### 3. Agents
- Dynamically decide which actions or tools to use.
- Adapt their execution based on intermediate results.
- Suitable for tasks requiring flexible decision-making.

### 4. Agentic Systems
- Combine LLMs, agents, workflows, tools, memory, and supporting infrastructure.
- Coordinate multiple components to accomplish complex objectives.

**Design Principle:** Use deterministic workflows wherever possible. Introduce autonomous decision-making only when the task requires it.

---

## 3. LLM LIMITATIONS

Understanding model limitations is essential when designing reliable AI systems.

### 1. Hallucinations
- Generate plausible but factually incorrect information.
- May present unsupported claims with confidence.

### 2. Stale Knowledge
- Pretrained knowledge may not reflect recent developments.
- External retrieval is necessary when current information matters.

### 3. Context Limits
- Models have finite context windows.
- Large inputs can increase cost and latency.
- Important information may be overlooked or lost in long contexts.

### 4. Information Integrity
- Retrieved information may be incomplete, outdated, contradictory, or untrusted.
- Incorrect source material can produce incorrect answers.

### 5. Safety and Bias
- Outputs may contain unsafe instructions or biased information.
- Systems need appropriate validation and safeguards.

---

## 4. BENEFITS OF CONTEXT ENGINEERING

Effective context engineering helps:

- Improve grounding and reduce avoidable hallucinations.
- Increase task-specific consistency and reliability.
- Optimize token consumption, latency, and inference cost.
- Organize instructions, memory, tools, and workflow state.
- Improve evaluation, observability, and debugging.
- Control which information reaches the model and when.

---

## 5. SIX SOURCES OF CONTEXT

Context is not limited to retrieved documents. It consists of multiple information sources.

### 1. Instructions
**Purpose:** Define rules, behavior, and constraints.

Examples:
- System prompts
- Developer instructions
- Policies
- Output schemas

### 2. External Knowledge
**Purpose:** Supply factual information unavailable or insufficiently represented in model knowledge.

Examples:
- Documents
- Databases
- APIs
- Websites
- Knowledge graphs

### 3. Tools
**Purpose:** Enable external actions, computation, and information retrieval.

Examples:
- Web search
- Python execution
- Database queries
- External APIs

### 4. Memory
**Purpose:** Retain useful information across interactions.

Examples:
- User preferences
- Previous decisions
- Historical interactions
- Long-term project information

### 5. State
**Purpose:** Track the current execution status of a system.

Examples:
- Current workflow step
- Execution history
- Retry count
- Pending tasks
- Intermediate results

### 6. User Prompts
**Purpose:** Express the user's intent and requirements.

Examples:
- Questions
- Goals
- Constraints
- Task-specific instructions

---

## 6. EXTERNAL KNOWLEDGE

**Definition:**

External knowledge provides information that may be missing, incomplete, or outdated in the model's pretrained knowledge.

### Common Sources
- Documents and files
- Relational databases
- APIs
- Websites
- Knowledge graphs

### Primary Objective
Supply relevant, current, and verifiable information to improve the accuracy and grounding of model responses.

**Important:** External knowledge is only useful when the system retrieves relevant information and preserves its source integrity.

---

## 7. RETRIEVAL-AUGMENTED GENERATION (RAG)

**Definition:**

RAG is an architecture that retrieves relevant external information and supplies it to an LLM as additional context before generation.

### RAG Pipeline

1. Document Ingestion
2. Document Processing
3. Chunking
4. Embedding Generation
5. Vector Storage and Indexing
6. Query Processing
7. Retrieval
8. Reranking (Optional)
9. Context Construction
10. LLM Generation
11. Evaluation

### Key Architectural Decisions

- Chunk size and overlap
- Embedding model selection
- Vector database selection
- Indexing strategy
- Retrieval method
- Top-K selection
- Metadata filtering
- Hybrid search
- Reranking
- Context assembly
- Citation handling

### Key Insight

Strong generation cannot reliably compensate for missing, irrelevant, or incorrect evidence.

Retrieval quality directly influences the quality of grounded responses.

---

## 8. KNOWLEDGE GRAPHS AND GRAPH RAG

### Knowledge Graph

A knowledge graph represents information through entities and their relationships.

It typically consists of:

- Nodes: Entities or concepts.
- Edges: Relationships between entities.
- Properties: Attributes associated with nodes or edges.

### Graph RAG

Graph RAG uses graph structures and relationships to retrieve relevant information and support LLM generation.

### Best Suited For

- Multi-hop questions
- Relationship analysis
- Connected knowledge exploration
- Questions involving relationships across multiple entities

### Trade-offs

- Graph construction complexity
- Entity resolution
- Relationship extraction
- Ongoing maintenance
- Additional storage and query complexity

**Key Insight:** Graph-based retrieval is valuable when relationships between information are important, not simply because a graph is a more advanced data structure.

---

## 9. AGENTIC RAG

**Definition:**

Agentic RAG uses an agent to dynamically plan, execute, evaluate, and refine retrieval operations instead of relying exclusively on a fixed retrieval pipeline.

### Typical Process

1. Understand the Query
2. Plan Retrieval
3. Execute Search
4. Evaluate Retrieved Evidence
5. Identify Information Gaps
6. Refine or Repeat Retrieval
7. Synthesize the Final Response

### Best Suited For

- Complex research tasks
- Multi-source information analysis
- Questions requiring iterative searches
- Tasks where the retrieval strategy depends on intermediate results

### Trade-offs

- Higher latency
- Higher token consumption
- Increased inference cost
- More complex control flow
- Potential tool-selection and execution failures

**Design Principle:** Use adaptive retrieval only when its improvement in answer quality justifies the additional operational overhead.

---

## 10. VECTORLESS RETRIEVAL

**Definition:**

Vectorless retrieval refers to approaches that retrieve information without relying primarily on embedding-based vector similarity.

### Common Methods

- Keyword search
- BM25 retrieval
- Exact-match search
- Document structure
- Headings and sections
- Metadata filtering
- SQL queries
- Deterministic lookups

### Best Suited For

- Exact identifiers
- Structured documents
- Financial statements
- Specific values and dates
- Predictable information lookups

### Limitations

- May miss semantically related information expressed using different terminology.
- Keyword matching alone may not capture contextual meaning.

**Key Insight:** Vectorless retrieval and vector-based retrieval can complement each other through hybrid search.

---

## 11. CACHE-AUGMENTED GENERATION (CAG)

**Definition:**

Cache-Augmented Generation refers to approaches that prepare or reuse knowledge and computation to reduce repeated processing during LLM inference.

### Common Implementations

- Preloaded context
- Reusable prompt prefixes
- Model KV-cache reuse
- Cached retrieval results
- Cached intermediate computation

### Best Suited For

- Repeated queries over stable knowledge
- Applications with recurring prompt structures
- Workloads where repeated context processing creates unnecessary overhead

### Benefits

- Reduces repeated retrieval or prefill work.
- Can improve response latency.
- Reduces redundant computation.

### Trade-offs

- Memory consumption
- Cache invalidation
- Data freshness
- Cache consistency
- Implementation complexity

**Important Distinction:**

Caching does not permanently teach the model new knowledge or modify its model weights.

The cache stores reusable context or computation rather than updating the underlying model.

---

## 12. CONTEXT STUFFING

**Definition:**

Context stuffing is an approach where the entire relevant document or information source is supplied directly to the model rather than retrieving selected chunks.

### Best Suited For

- Short documents
- Whole-document analysis
- Broad summarization
- Tasks requiring document-wide context

### Benefits

- Preserves broader document context.
- Avoids some retrieval misses.
- Reduces the need for complex retrieval infrastructure.

### Trade-offs

- Higher token consumption
- Increased inference cost
- Higher latency
- Context-window pressure
- Potential distraction from irrelevant information

**Key Insight:** Supplying more information does not automatically improve answer quality. Relevance and information organization remain important.

---

## 13. COMPARISON OF RETRIEVAL APPROACHES

| Approach | Mechanism | Suitable Use Case |
|---|---|---|
| Traditional RAG | Retrieves relevant chunks | Large knowledge bases |
| Graph RAG | Uses entities and relationships | Multi-hop reasoning |
| Agentic RAG | Dynamically plans retrieval | Complex research |
| Vectorless Retrieval | Uses keywords, structure, and metadata | Exact and structured lookups |
| CAG | Reuses prepared context or cached computation | Repeated queries over stable context |
| Context Stuffing | Supplies the entire relevant source | Small documents and whole-document analysis |

### Important Note

These approaches are not mutually exclusive.

A production system can combine multiple retrieval strategies depending on:

- Workload characteristics
- Query complexity
- Quality requirements
- Latency constraints
- Cost limitations
- Operational complexity

---

## 14. CHOOSING THE RIGHT APPROACH

Architecture selection should begin with the actual requirements of the task.

| Requirement | Starting Approach |
|---|---|
| Search a large document collection | Traditional RAG |
| Answer relationship-heavy questions | Graph RAG |
| Research across multiple sources adaptively | Agentic RAG |
| Find exact values or structured sections | Vectorless Retrieval |
| Serve repeated queries over stable context | Caching / CAG |
| Analyze a small document in its entirety | Context Stuffing |

### Architecture Selection Principle

These are starting points, not universal rules.

Benchmark alternative approaches against representative queries before committing to an architecture.

---

## 15. EVALUATION CRITERIA

A context engineering system should be evaluated across multiple dimensions.

### 1. Answer Quality
- Correctness
- Relevance
- Completeness

### 2. Faithfulness
- Whether generated claims are supported by available evidence.
- Whether the response introduces unsupported information.

### 3. Retrieval Quality
- Recall
- Precision
- Ranking effectiveness
- Context relevance

### 4. Freshness
- Whether retrieved information reflects current data.
- Whether outdated information is identified or excluded.

### 5. Efficiency
- Token consumption
- End-to-end latency
- Inference cost
- Cost per successful task

### 6. Reliability
- Behavior under missing information.
- Handling conflicting sources.
- Handling malformed inputs.
- Recovery from retrieval and tool failures.

### 7. Operational Complexity
- Maintenance requirements
- Observability
- Monitoring
- Failure handling
- Scalability

---

## 16. FINAL TAKEAWAYS

1. Context engineering determines what information the model receives, how it is organized, and when it is supplied.

2. Context includes instructions, external knowledge, tools, memory, state, and user prompts.

3. Different retrieval architectures solve different information-access problems.

4. More context is not necessarily better. Relevance, quality, and organization matter more than raw context volume.

5. No single retrieval architecture is optimal for every workload.

6. Deterministic retrieval and workflows should be preferred when they adequately solve the problem.

7. Adaptive retrieval and agentic behavior should be introduced when their benefits justify their additional complexity.

8. Caching can reduce repeated computation but introduces invalidation and freshness challenges.

9. Evaluation must cover answer quality, faithfulness, retrieval performance, efficiency, and reliability.

10. Production architecture decisions should be driven by measurable results rather than architectural complexity.

---

## CORE PRINCIPLE

> Provide the most relevant and trustworthy context for the task while balancing answer quality, latency, cost, and operational complexity.

**Context Engineering is not about giving an LLM more information. It is about giving it the right information, in the right structure, at the right time.**