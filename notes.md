CONTEXT ENGINEERING — QUICK REFERENCE
======================================

A concise reference covering context engineering, retrieval strategies, and LLM system design.

1. DEFINITION
-------------

Context Engineering:
The discipline of selecting, structuring, and managing the information supplied to an LLM at each step.

Goal:
Maximize answer quality and reliability while controlling:
- Token usage
- Latency
- Cost
- Complexity

2. LLM SYSTEM ARCHITECTURE
--------------------------

1. LLM Calls
   - Direct input-output interactions.

2. Workflows
   - Execute predefined steps.

3. Agents
   - Dynamically select actions based on the task.

4. Agentic Systems
   - Combine agents, tools, workflows, and supporting components.

3. LLM LIMITATIONS
------------------

1. Hallucinations
   - Generate plausible but incorrect information.

2. Stale Knowledge
   - May lack recent or updated information.

3. Context Limits
   - Cannot process unlimited information in one context window.

4. Information Integrity
   - Retrieved data may be incomplete, conflicting, or untrusted.

5. Safety and Bias
   - Outputs may contain unsafe instructions or biased information.

4. BENEFITS OF CONTEXT ENGINEERING
----------------------------------

- Improves grounding and reduces avoidable hallucinations.
- Increases task-specific consistency and reliability.
- Controls token usage, latency, and inference cost.
- Organizes memory, tools, and workflow state.
- Improves evaluation, observability, and debugging.

5. SIX SOURCES OF CONTEXT
-------------------------

1. Instructions
   Purpose: Define rules and constraints.
   Examples: System prompts, policies.

2. External Knowledge
   Purpose: Supply factual information.
   Examples: Documents, databases, APIs.

3. Tools
   Purpose: Enable actions and computation.
   Examples: Search, Python, API calls.

4. Memory
   Purpose: Retain useful information across interactions.
   Examples: User preferences, past decisions.

5. State
   Purpose: Track current execution status.
   Examples: Workflow step, retries, pending tasks.

6. User Prompts
   Purpose: Express intent and requirements.
   Examples: Questions, goals, constraints.

6. EXTERNAL KNOWLEDGE
---------------------

Definition:
External knowledge supplies information that may be missing from or newer than the model's training data.

Sources:
- Documents
- Databases
- APIs
- Websites
- Knowledge graphs

Goal:
Ground responses in relevant, current, and verifiable information.

7. RETRIEVAL-AUGMENTED GENERATION (RAG)
---------------------------------------

Definition:
RAG retrieves external information and supplies it to an LLM to improve grounding.

Pipeline:

Indexing
   ↓
Chunking
   ↓
Embedding
   ↓
Storage
   ↓
Retrieval
   ↓
Context Construction
   ↓
Generation
   ↓
Evaluation

Key Design Decisions:
- Chunk size and overlap
- Embedding model
- Vector database and indexing strategy
- Retrieval method and top-k
- Metadata filtering and reranking
- Context assembly and citation handling

Key Insight:
Strong generation cannot reliably compensate for missing or irrelevant evidence.

8. KNOWLEDGE GRAPHS AND GRAPH RAG
---------------------------------

Knowledge Graph:
Represents entities and their relationships as connected data.

Graph RAG:
Uses graph structures and relationships to retrieve relevant information for generation.

Best Suited For:
- Multi-hop questions
- Relationship analysis
- Connected knowledge

Trade-offs:
- Graph construction
- Entity resolution
- Ongoing maintenance
- Additional system complexity

9. AGENTIC RAG
--------------

Definition:
Agentic RAG allows an agent to plan and adapt retrieval instead of following only a fixed retrieval pipeline.

Process:

Understand
   ↓
Plan
   ↓
Retrieve
   ↓
Evaluate Evidence
   ↓
Refine Search
   ↓
Synthesize

Best Suited For:
- Complex research
- Multiple-source analysis
- Iterative searches

Trade-offs:
- Higher latency
- Higher token cost
- Potential failure risk

Design Principle:
Use adaptive retrieval only when its quality improvements justify the overhead.

10. VECTORLESS RAG
------------------

Definition:
Retrieves information without relying primarily on embedding-based vector similarity.

Methods:
- Keyword search
- Document structure
- Headings
- Metadata
- Deterministic lookup

Best Suited For:
- Exact identifiers
- Structured documents
- Predictable lookups

Limitation:
May miss semantically related information expressed using different terminology.

11. CACHE-AUGMENTED GENERATION (CAG)
-----------------------------------

Definition:
Approaches that prepare or cache knowledge and computation to reduce repeated processing.

Implementations may reuse:
- Preloaded context
- Model KV-cache state
- Cached computation

Best Suited For:
Repeated queries over stable knowledge.

Benefits:
- Reduces repeated retrieval or prefill work.
- Can improve latency.

Trade-offs:
- Memory consumption
- Cache invalidation
- Data freshness
- Implementation complexity

Important:
Caching does not permanently teach the model new knowledge or update its weights.

12. CONTEXT STUFFING
--------------------

Definition:
Places an entire relevant document or source directly into the model's context window rather than retrieving selected chunks.

Best Suited For:
- Short documents
- Whole-document analysis
- Broad summarization

Benefits:
- Preserves document-wide context.
- Avoids some retrieval misses.

Trade-offs:
- Higher token usage
- Higher cost
- Higher latency
- Context-window pressure

13. COMPARISON OF RETRIEVAL APPROACHES
-------------------------------------

1. Traditional RAG
   Mechanism: Retrieves relevant chunks.
   Use Case: Large knowledge bases.

2. Graph RAG
   Mechanism: Uses entities and relationships.
   Use Case: Multi-hop reasoning.

3. Agentic RAG
   Mechanism: Dynamically plans retrieval.
   Use Case: Complex research.

4. Vectorless Retrieval
   Mechanism: Uses keywords, structure, or metadata.
   Use Case: Exact and structured lookup.

5. CAG
   Mechanism: Reuses prepared context or cached computation.
   Use Case: Repeated queries over stable context.

6. Context Stuffing
   Mechanism: Supplies the full relevant source.
   Use Case: Small sources and whole-document analysis.

Note:
These approaches are not mutually exclusive. A system can combine them according to workload, quality requirements, and operational constraints.

14. CHOOSING AN APPROACH
------------------------

Requirement:
Search a large document collection.
Starting Point: Traditional RAG.

Requirement:
Answer relationship-heavy questions.
Starting Point: Graph RAG.

Requirement:
Research across multiple sources adaptively.
Starting Point: Agentic RAG.

Requirement:
Find exact values or structured sections.
Starting Point: Vectorless Retrieval.

Requirement:
Serve repeated queries over stable context.
Starting Point: Caching or CAG.

Requirement:
Analyze a small document in its entirety.
Starting Point: Context Stuffing.

Important:
These are starting points, not universal rules. Benchmark alternatives against representative queries before choosing an architecture.

15. EVALUATION CRITERIA
-----------------------

1. Answer Quality
   - Correctness
   - Relevance
   - Completeness

2. Faithfulness
   - Whether claims are supported by available evidence.

3. Retrieval Quality
   - Recall
   - Precision
   - Ranking effectiveness

4. Freshness
   - Whether context reflects current data.

5. Efficiency
   - Token usage
   - Latency
   - Cost per successful task

6. Reliability
   - Behavior under missing, conflicting, or malformed information.

7. Operational Complexity
   - Maintenance
   - Observability
   - Failure handling

FINAL TAKEAWAYS
---------------

1. Context engineering determines what information the model receives, how it is organized, and when it is supplied.

2. Context includes instructions, external knowledge, tools, memory, state, and user prompts.

3. Retrieval approaches have different strengths, costs, and failure modes.

4. More context is not always better; relevance and organization matter.

5. No single architecture is optimal for every workload.

6. Measure quality, latency, cost, and reliability before adding architectural complexity.

CORE PRINCIPLE
--------------

"Provide the most relevant and trustworthy context for the task while balancing answer quality, latency, cost, and operational complexity."