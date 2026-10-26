# Context Engineering — Chapter-by-Chapter Summary

## Chapter 1: Definition of Context Engineering

Context engineering is the discipline of structuring and managing all inputs that influence how an LLM interprets and responds to a task.

**Core objective:** Provide the right information at the right time while balancing relevance, token usage, and cost.

## Chapter 2: Levels of LLM System Architecture

1. **LLM Calls:** Direct input-output interaction with an LLM.
2. **AI Workflows:** Predefined steps control task execution.
3. **AI Agents:** An LLM dynamically selects actions and iterates through an agent loop.
4. **Agentic Systems:** Multiple agents or components coordinate within a broader runtime.

**Key distinction:** Workflows follow predefined paths; agents can dynamically choose actions.

## Chapter 3: Key Limitations of LLMs

- **Hallucinations:** Generating incorrect or unsupported information.
- **Stale Data:** Using outdated knowledge.
- **Limited Context Window:** Restrictions on the amount of information processed at once.
- **Information Integrity:** Difficulty maintaining trustworthy inputs and outputs.
- **Safety:** Risk of harmful or policy-violating outputs.
- **Bias:** Reflecting biases from training data or context.

## Chapter 4: Benefits of Context Engineering

- Reduces hallucinations through relevant, grounded information.
- Improves response consistency.
- Controls token consumption and inference costs.
- Helps manage growing system complexity.
- Improves production AI reliability.

## Chapter 5: Six Sources of Context

1. **Instructions:** Rules, constraints, and behavioral guidelines.
2. **External Knowledge:** Information from documents, databases, APIs, and other sources.
3. **Tools:** Available functions, commands, and tool outputs.
4. **Memory:** Relevant information retained across interactions.
5. **State:** Current task, workflow, or environment status.
6. **User Prompts:** The immediate request, intent, and requirements.

**Remember:** Context includes more than the prompt. It encompasses everything that influences the model's response.

## Chapter 6: External Knowledge

External knowledge provides information that may be missing from the LLM's training data or may have changed since training.

**Common sources:**
- Documents and PDFs
- Databases and knowledge bases
- APIs and live data
- Websites and search engines
- Knowledge graphs

**Purpose:** Ground LLM responses in relevant, current, domain-specific, or verifiable information.

## Chapter 7: Retrieval-Augmented Generation (RAG)

RAG combines information retrieval with LLM generation. It retrieves relevant external information and supplies it to the model as context.

### RAG Pipeline

1. **Indexing:** Load and process documents.
2. **Chunking:** Split documents into smaller sections.
3. **Embedding:** Convert chunks into numerical vectors.
4. **Storage:** Store vectors and metadata.
5. **Retrieval:** Find relevant information for a query.
6. **Context Construction:** Select and organize retrieved results.
7. **Generation:** Generate an answer using the query and retrieved context.
8. **Evaluation:** Measure retrieval quality, faithfulness, correctness, latency, and cost.

### Key Design Choices

- **Chunking:** Controls how information is divided.
- **Embeddings:** Represent semantic information numerically.
- **Retrieval Models:** Determine which information is selected.
- **Vector Databases:** Store and search vectors.
- **Reranking:** Reorders results by relevance.
- **Metadata Filtering:** Restricts results using attributes such as date or source.

**Key takeaway:** RAG quality depends heavily on retrieving the right information, not merely on the LLM generating a good response.

## Chapter 8: Knowledge Graphs and Graph RAG

### Knowledge Graphs

A knowledge graph represents information through entities and explicit relationships.

Example:

`Company A → acquired → Company B`

### Graph RAG

Graph RAG retrieves relevant entities, relationships, and connected information from a knowledge graph to provide context to an LLM.

**Useful for:**
- Multi-hop questions
- Relationship analysis
- Connecting information across documents
- Understanding complex networks of entities

**Trade-off:** Graph construction, entity resolution, and maintenance increase system complexity and cost.

## Chapter 9: Agentic RAG

Agentic RAG uses an AI agent to dynamically control retrieval instead of relying entirely on a fixed retrieval pipeline.

### Typical Process

1. Understand the question.
2. Plan a retrieval strategy.
3. Select a tool or information source.
4. Retrieve information.
5. Evaluate whether the results are sufficient.
6. Search again if necessary.
7. Synthesize the evidence into an answer.

**Useful for:** Complex research, multi-source investigation, and tasks requiring iterative searches.

**Trade-off:** Greater flexibility can introduce additional latency, token costs, tool failures, and reasoning errors.

**Decision rule:** Use agentic retrieval when adaptive search decisions provide enough value to justify the added complexity.

## Chapter 10: Vectorless RAG

Vectorless RAG retrieves information without depending on embedding-based vector similarity.

It can use:
- Document hierarchies
- Headings and tables of contents
- Metadata and document identifiers
- Keywords and structural relationships

Example:

`Document → Chapter → Section → Subsection`

**Useful for:** Structured documents, exact lookups, and workloads where document structure provides reliable retrieval signals.

**Trade-off:** It may struggle when relevant information is expressed using different wording or spread across unrelated sections.

**Key takeaway:** Vectorless retrieval uses document structure or other non-vector methods instead of relying on embeddings.

## Chapter 11: Cache-Augmented Generation (CAG)

CAG reduces repeated retrieval work by preparing knowledge context in advance and, in supported implementations, reusing the model's key-value (KV) cache.

### Typical Process

1. Load a known knowledge collection.
2. Prepare the context in advance.
3. Prefill the model with the knowledge.
4. Reuse the prepared context or KV cache when supported.
5. Generate responses without repeating the entire retrieval process.

**Useful for:**
- Stable knowledge collections
- Repeated queries over the same information
- Workloads where retrieval latency is a bottleneck

**Trade-off:** Large contexts consume memory, and knowledge updates require careful cache management.

**Important:** CAG does not permanently teach the model new knowledge. It makes prepared context or cached computation reusable.

## Chapter 12: Context Stuffing

Context stuffing places the entire relevant document or a large portion of it directly into the LLM's context window instead of first retrieving selected chunks.

### Typical Process

1. Load the document.
2. Convert it into a model-readable format.
3. Insert the content into the prompt.
4. Ask the LLM to analyze the supplied information.

**Useful for:**
- Short documents
- Small document collections
- Tasks requiring a broad view of the source
- Prototypes where a retrieval pipeline is unnecessary

**Trade-offs:**
- Higher input-token consumption
- Potentially higher latency and cost
- Context-window limitations
- Risk of important information being overlooked

**Decision rule:** Use context stuffing when the relevant material fits comfortably within the context window and the cost is acceptable.

## Chapter 13: Comparison of External Knowledge Approaches

| Approach | Core mechanism | Best suited for |
|---|---|---|
| Traditional RAG | Retrieves relevant document chunks | Large knowledge bases |
| Graph RAG | Traverses entities and relationships | Relationship-heavy questions |
| Agentic RAG | Dynamically plans and executes retrieval | Complex research |
| Vectorless RAG | Uses structure, metadata, or keywords | Structured documents |
| CAG | Reuses prepared knowledge context or cached computation | Repeated queries over stable knowledge |
| Context Stuffing | Loads the full relevant source into the prompt | Small document collections |

These approaches are not mutually exclusive. A production system can combine retrieval, graph traversal, caching, and metadata filtering.

## Chapter 14: Choosing the Right Approach

- **Traditional RAG:** Choose when the knowledge base is large and only relevant subsets should enter the context.
- **Graph RAG:** Choose when relationships between entities are essential.
- **Agentic RAG:** Choose when retrieval requires dynamic decisions and multiple searches.
- **Vectorless RAG:** Choose when document structure or metadata supports reliable retrieval.
- **CAG:** Choose when knowledge is stable and repeated queries make context reuse beneficial.
- **Context Stuffing:** Choose when the relevant source is small enough to include directly.

Evaluate each design using:
- Answer quality and faithfulness
- Retrieval accuracy
- Data freshness
- Context-window usage
- Latency
- Inference and infrastructure cost
- Operational complexity

## Final Chapter: Key Takeaways

1. Context engineering is about selecting and organizing the right information, not simply adding more context.
2. An LLM's output depends on the instructions, knowledge, tools, memory, state, and prompt available to it.
3. RAG retrieves relevant external information to ground generation.
4. Graph RAG uses relationships; Agentic RAG adapts retrieval dynamically.
5. Vectorless RAG uses non-vector retrieval mechanisms.
6. CAG reduces repeated work through prepared context or reusable cached computation.
7. Context stuffing supplies the full relevant source directly when practical.
8. No single approach is best for every application.

**Core principle:** Provide sufficient, relevant, trustworthy context while balancing quality, latency, cost, and system complexity.