# CONTEXT ENGINEERING — TECHNICAL REFERENCE

**Topics:** LLM System Design · Context Engineering · Retrieval Architectures · Production AI Systems

---

# 1. FOUNDATION

## 1.1 What is Context Engineering?

Context Engineering is the discipline of selecting, structuring, managing, and delivering the right information to an LLM at each stage of execution.

**Primary Objective:**

Maximize answer quality and system reliability while optimizing:

- Token consumption
- Latency
- Inference cost
- Operational complexity
- Information relevance

**Core Idea:** The quality of an LLM system depends not only on the model but also on the quality of the context supplied to it.

## 1.2 LLM System Architecture

LLM-based systems can be divided into four categories:

| Category | Description | Best Suited For |
|---|---|---|
| LLM Calls | Direct input-output interactions | Straightforward tasks |
| Workflows | Predetermined sequences of operations | Predictable tasks |
| Agents | Dynamically select tools and actions | Flexible decision-making |
| Agentic Systems | Coordinate agents, workflows, tools and memory | Complex objectives |

**Design Principle:** Prefer deterministic workflows wherever possible. Introduce autonomous decision-making only when necessary.

## 1.3 LLM Limitations

| Limitation | Description |
|---|---|
| Hallucinations | Plausible but factually incorrect outputs |
| Stale Knowledge | Pretrained knowledge may not reflect recent developments |
| Context Limits | Finite context windows and increasing token costs |
| Information Integrity | Retrieved information may be incomplete or contradictory |
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

**Important Distinction:**

- Memory represents information retained across interactions.
- State represents the current execution status.
- External knowledge provides information from outside the model.

---

# 3. EXTERNAL KNOWLEDGE AND RETRIEVAL

## 3.1 External Knowledge

External knowledge provides information that may be missing, incomplete, or outdated in the model's pretrained knowledge.

**Common Sources:**

- Documents and files
- Relational databases
- APIs
- Websites
- Knowledge graphs

**Objective:** Supply relevant, current, and verifiable information to improve response accuracy and grounding.

## 3.2 Retrieval-Augmented Generation (RAG)

**Definition:** RAG retrieves relevant external information and supplies it to an LLM as additional context before generation.

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
- Embedding model
- Vector database
- Indexing strategy
- Retrieval method
- Top-K selection
- Metadata filtering
- Hybrid search
- Reranking
- Context assembly
- Citation handling

**Key Insight:** Strong generation cannot reliably compensate for missing, irrelevant, or incorrect evidence. Retrieval quality directly influences grounded response quality.

---

# 4. ADVANCED RETRIEVAL ARCHITECTURES

## 4.1 Knowledge Graphs and Graph RAG

### Knowledge Graph

A knowledge graph represents information through entities and their relationships.

Its components:

- Nodes — Entities or concepts.
- Edges — Relationships between entities.
- Properties — Attributes associated with nodes or edges.

### Graph RAG

Graph RAG uses graph structures and relationships to retrieve information and support LLM generation.

**Best Suited For:**

- Multi-hop questions
- Relationship analysis
- Connected knowledge exploration
- Questions involving multiple related entities

**Trade-offs:**

- Graph construction complexity
- Entity resolution
- Relationship extraction
- Ongoing maintenance
- Additional storage and query complexity

**Design Principle:** Use graph-based retrieval when relationships between information matter, not simply because graphs are more advanced.

## 4.2 Agentic RAG

**Definition:** Agentic RAG uses an agent to dynamically plan, execute, evaluate, and refine retrieval operations instead of relying exclusively on a fixed retrieval pipeline.

### Typical Execution

1. Understand the Query
2. Plan Retrieval
3. Execute Search
4. Evaluate Retrieved Evidence
5. Identify Information Gaps
6. Refine or Repeat Retrieval
7. Synthesize Final Response

**Best Suited For:**

- Complex research tasks
- Multi-source information analysis
- Iterative searches
- Tasks where retrieval strategy depends on intermediate results

**Trade-offs:**

- Higher latency
- Higher token consumption
- Increased inference cost
- More complex control flow
- Potential tool-selection and execution failures

**Design Principle:** Use adaptive retrieval only when its improvement in answer quality justifies the additional operational overhead.

## 4.3 Vectorless Retrieval

**Definition:** Retrieval approaches that do not rely primarily on embedding-based vector similarity.

### Common Methods

- Keyword search
- BM25 retrieval
- Exact-match search
- Document structure
- Headings and sections
- Metadata filtering
- SQL queries
- Deterministic lookups

**Best Suited For:**

- Exact identifiers
- Structured documents
- Financial statements
- Specific values and dates
- Predictable information lookups

**Limitations:**

- May miss semantically related information expressed using different terminology.
- Keyword matching alone may not capture contextual meaning.

**Key Insight:** Vectorless retrieval and vector-based retrieval can complement each other through hybrid search.

## 4.4 Cache-Augmented Generation (CAG)

**Definition:** Approaches that prepare or reuse knowledge and computation to reduce repeated processing during LLM inference.

### Common Implementations

- Preloaded context
- Reusable prompt prefixes
- Model KV-cache reuse
- Cached retrieval results
- Cached intermediate computation

**Best Suited For:**

- Repeated queries over stable knowledge
- Applications with recurring prompt structures
- Workloads where repeated context processing creates unnecessary overhead

**Benefits:**

- Reduces repeated retrieval or prefill work.
- Can improve response latency.
- Reduces redundant computation.

**Trade-offs:**

- Memory consumption
- Cache invalidation
- Data freshness
- Cache consistency
- Implementation complexity

**Important Distinction:** Caching does not permanently teach the model new knowledge or modify its model weights. The cache stores reusable context or computation rather than updating the underlying model.

## 4.5 Context Stuffing

**Definition:** Supplying the entire relevant document or information source directly to the model rather than retrieving selected chunks.

**Best Suited For:**

- Short documents
- Whole-document analysis
- Broad summarization
- Tasks requiring document-wide context

**Benefits:**

- Preserves broader document context.
- Avoids some retrieval misses.
- Reduces the need for complex retrieval infrastructure.

**Trade-offs:**

- Higher token consumption
- Increased inference cost
- Higher latency
- Context-window pressure
- Potential distraction from irrelevant information

**Key Insight:** More information does not automatically improve answer quality. Relevance and information organization remain important.

---

# 5. RETRIEVAL ARCHITECTURE COMPARISON

| Approach | Mechanism | Suitable Use Case |
|---|---|---|
| Traditional RAG | Retrieves relevant chunks | Large knowledge bases |
| Graph RAG | Uses entities and relationships | Multi-hop reasoning |
| Agentic RAG | Dynamically plans retrieval | Complex research |
| Vectorless Retrieval | Keywords, structure and metadata | Exact lookups |
| CAG | Reuses context or computation | Repeated queries |
| Context Stuffing | Supplies entire relevant source | Small documents |

**Important Note:** These approaches are not mutually exclusive.

A production system can combine multiple retrieval strategies depending on:

- Workload characteristics
- Query complexity
- Quality requirements
- Latency constraints
- Cost limitations
- Operational complexity

---

# 6. CHOOSING THE RIGHT ARCHITECTURE

Architecture selection should begin with the actual requirements of the task.

| Requirement | Starting Approach |
|---|---|
| Search large document collections | Traditional RAG |
| Answer relationship-heavy questions | Graph RAG |
| Research across multiple sources adaptively | Agentic RAG |
| Find exact values or structured sections | Vectorless Retrieval |
| Serve repeated queries over stable context | Caching / CAG |
| Analyze a small document in its entirety | Context Stuffing |

### Architecture Selection Principle

These are starting points, not universal rules.

Benchmark alternative approaches against representative queries before committing to an architecture.

---

# 7. EVALUATION AND RELIABILITY

A context engineering system should be evaluated across seven dimensions.

| Dimension | What to Evaluate |
|---|---|
| Answer Quality | Correctness, relevance, completeness |
| Faithfulness | Whether claims are supported by evidence |
| Retrieval Quality | Recall, precision, ranking effectiveness |
| Freshness | Whether information reflects current data |
| Efficiency | Tokens, latency, inference cost |
| Reliability | Missing information, conflicting sources, failures |
| Operational Complexity | Maintenance, observability, monitoring, scalability |

### Evaluation Questions

Before deploying a system, ask:

1. Does the system retrieve the correct information?
2. Does the LLM generate answers grounded in retrieved evidence?
3. Can the system identify missing or conflicting information?
4. Does it handle malformed inputs?
5. What happens when retrieval or tools fail?
6. What is the cost per successful task?
7. Can failures be traced and debugged?

---

# 8. FINAL TAKEAWAYS

1. Context engineering determines what information the model receives, how it is organized, and when it is supplied.
2. Context includes instructions, external knowledge, tools, memory, state, and user prompts.
3. Different retrieval architectures solve different information-access problems.
4. More context is not necessarily better. Relevance and organization matter more than raw volume.
5. No single retrieval architecture is optimal for every workload.
6. Prefer deterministic retrieval and workflows when they adequately solve the problem.
7. Introduce adaptive retrieval and agentic behavior when their benefits justify additional complexity.
8. Caching reduces repeated computation but introduces invalidation and freshness challenges.
9. Evaluation must cover answer quality, faithfulness, retrieval performance, efficiency, and reliability.
10. Production architecture decisions should be driven by measurable results rather than architectural complexity.

---

# CORE PRINCIPLE

> Provide the most relevant and trustworthy context for the task while balancing answer quality, latency, cost, and operational complexity.

**Context Engineering is not about giving an LLM more information. It is about giving it the right information, in the right structure, at the right time.**