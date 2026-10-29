# Context Engineering — Quick Reference

A concise reference to context engineering, retrieval strategies, and LLM system design.

## 1. Definition

**Context engineering** is the discipline of selecting, structuring, and managing the information supplied to an LLM at each step.

**Goal:** Maximize answer quality and reliability while controlling token usage, latency, cost, and complexity.

## 2. LLM System Architecture

| Architecture | Description |
|---|---|
| LLM Calls | Direct input-output interactions |
| Workflows | Execute predefined steps |
| Agents | Dynamically select actions based on the task |
| Agentic Systems | Combine agents, tools, workflows, and supporting components |

## 3. LLM Limitations

- **Hallucinations:** Generate plausible but incorrect information.
- **Stale Knowledge:** May lack recent or updated information.
- **Context Limits:** Cannot process unlimited information in one context window.
- **Information Integrity:** Retrieved data may be incomplete, conflicting, or untrusted.
- **Safety and Bias:** Outputs may contain unsafe instructions or biased information.

## 4. Benefits of Context Engineering

- Improves grounding and reduces avoidable hallucinations.
- Increases task-specific consistency and reliability.
- Controls token usage, latency, and inference cost.
- Organizes memory, tools, and workflow state.
- Improves evaluation, observability, and debugging.

## 5. Six Sources of Context

| Source | Purpose | Examples |
|---|---|---|
| Instructions | Define rules and constraints | System prompts, policies |
| External Knowledge | Supply factual information | Documents, databases, APIs |
| Tools | Enable actions and computation | Search, Python, API calls |
| Memory | Retain useful information across interactions | User preferences, past decisions |
| State | Track current execution status | Workflow step, retries, pending tasks |
| User Prompts | Express intent and requirements | Questions, goals, constraints |

## 6. External Knowledge

External knowledge supplies information that may be missing from or newer than the model's training data.

**Sources:** Documents, databases, APIs, websites, and knowledge graphs.

**Goal:** Ground responses in relevant, current, and verifiable information.

## 7. Retrieval-Augmented Generation (RAG)

RAG retrieves external information and supplies it to an LLM to improve grounding.

**Pipeline:**

Indexing → Chunking → Embedding → Storage → Retrieval → Context Construction → Generation → Evaluation

**Key design decisions:**
- Chunk size and overlap
- Embedding model
- Vector database and indexing strategy
- Retrieval method and top-k
- Metadata filtering and reranking
- Context assembly and citation handling

**Key insight:** Strong generation cannot reliably compensate for missing or irrelevant evidence.

## 8. Knowledge Graphs and Graph RAG

**Knowledge Graph:** Represents entities and their relationships as connected data.

**Graph RAG:** Uses graph structures and relationships to retrieve relevant information for generation.

- **Best suited for:** Multi-hop questions, relationship analysis, and connected knowledge.
- **Trade-offs:** Graph construction, entity resolution, and ongoing maintenance add complexity.

## 9. Agentic RAG

Agentic RAG allows an agent to plan and adapt retrieval instead of following only a fixed retrieval pipeline.

**Process:**

Understand → Plan → Retrieve → Evaluate Evidence → Refine Search → Synthesize

- **Best suited for:** Complex research requiring multiple sources or iterative searches.
- **Trade-offs:** Higher latency, token cost, and potential failure risk.
- **Design principle:** Use adaptive retrieval only when its quality improvements justify the overhead.

## 10. Vectorless RAG

Vectorless retrieval finds information without relying primarily on embedding-based vector similarity.

**Methods:** Keyword search, document structure, headings, metadata, and deterministic lookup.

- **Best suited for:** Exact identifiers, structured documents, and predictable lookups.
- **Limitation:** May miss semantically related information expressed using different terminology.

## 11. Cache-Augmented Generation (CAG)

CAG refers to approaches that prepare or cache knowledge and computation to reduce repeated processing. Implementations may reuse preloaded context or model KV-cache state.

- **Best suited for:** Repeated queries over stable knowledge.
- **Benefits:** Can reduce repeated retrieval or prefill work and improve latency.
- **Trade-offs:** Memory consumption, cache invalidation, freshness, and implementation complexity.

**Important:** Caching does not permanently teach the model new knowledge or update its weights.

## 12. Context Stuffing

Context stuffing places an entire relevant document or source directly into the model's context window rather than retrieving selected chunks.

- **Best suited for:** Short documents, whole-document analysis, and broad summarization.
- **Benefits:** Preserves document-wide context and avoids some retrieval misses.
- **Trade-offs:** Higher token usage, cost, latency, and context-window pressure.

## 13. Comparing Retrieval Approaches

| Approach | Core Mechanism | Best Suited For |
|---|---|---|
| Traditional RAG | Retrieves relevant chunks | Large knowledge bases |
| Graph RAG | Uses entities and relationships | Multi-hop reasoning |
| Agentic RAG | Dynamically plans retrieval | Complex research |
| Vectorless Retrieval | Uses keywords, structure, or metadata | Exact and structured lookup |
| CAG | Reuses prepared context or cached computation | Repeated queries over stable context |
| Context Stuffing | Supplies the full relevant source | Small sources and whole-document analysis |

**Note:** These approaches are not mutually exclusive. A system can combine them according to workload, quality requirements, and operational constraints.

## 14. Choosing an Approach

| Requirement | Starting Point |
|---|---|
| Search a large document collection | Traditional RAG |
| Answer relationship-heavy questions | Graph RAG |
| Research across multiple sources adaptively | Agentic RAG |
| Find exact values or structured sections | Vectorless Retrieval |
| Serve repeated queries over stable context | Caching or CAG |
| Analyze a small document in its entirety | Context Stuffing |

These are starting points, not universal rules. Benchmark alternatives against representative queries before choosing an architecture.

## 15. Evaluation Criteria

Evaluate each approach using:

- **Answer Quality:** Correctness, relevance, and completeness.
- **Faithfulness:** Whether claims are supported by available evidence.
- **Retrieval Quality:** Recall, precision, and ranking effectiveness.
- **Freshness:** Whether the context reflects current data.
- **Efficiency:** Token usage, latency, and cost per successful task.
- **Reliability:** Behavior under missing, conflicting, or malformed information.
- **Operational Complexity:** Maintenance, observability, and failure handling.

## Final Takeaways

- Context engineering determines what information the model receives, how it is organized, and when it is supplied.
- Context includes instructions, external knowledge, tools, memory, state, and user prompts.
- Retrieval approaches have different strengths, costs, and failure modes.
- More context is not always better; relevance and organization matter.
- No single architecture is optimal for every workload.
- Measure quality, latency, cost, and reliability before adding architectural complexity.

## Core Principle

> Provide the most relevant and trustworthy context for the task while balancing answer quality, latency, cost, and operational complexity.
