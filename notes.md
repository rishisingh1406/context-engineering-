# Context Engineering — Chapter-by-Chapter Notes

## Chapter 1: Definition of Context Engineering

- **Definition:** Context engineering is the discipline of structuring and managing all inputs that influence how an LLM interprets and responds to a task.
- **Core objective:** Provide the right information at the right time while balancing relevance, token usage, and cost.

## Chapter 2: Levels of LLM System Architecture

- **LLM Calls:** Direct input-output interaction with an LLM.
- **AI Workflows:** Predefined steps control task execution.
- **AI Agents:** An LLM dynamically selects actions and iterates through an agent loop.
- **Agentic Systems:** Multiple agents or components coordinate within a broader runtime.

### Key Distinction

- **Workflows:** Follow predefined execution paths.
- **Agents:** Can dynamically choose actions based on the task and available information.

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

### 1. Instructions
- Rules, constraints, and behavioral guidelines.

### 2. External Knowledge
- Information from documents, databases, APIs, and other sources.

### 3. Tools
- Available functions, commands, and tool outputs.

### 4. Memory
- Relevant information retained across interactions.

### 5. State
- Current task, workflow, or environment status.

### 6. User Prompts
- The immediate request, intent, and requirements.

### Key Takeaway

- Context includes more than the prompt.
- It encompasses everything that influences the model's response.

## Chapter 6: External Knowledge

- **Definition:** External knowledge provides information that may be missing from the LLM's training data or may have changed since training.

### Common Sources

- Documents and PDFs
- Databases and knowledge bases
- APIs and live data
- Websites and search engines
- Knowledge graphs

### Purpose

- Ground LLM responses in relevant, current, domain-specific, or verifiable information.

## Chapter 7: Retrieval-Augmented Generation (RAG)

- **Definition:** RAG combines information retrieval with LLM generation.
- It retrieves relevant external information and supplies it to the model as context.

### 7.1 RAG Pipeline

1. **Indexing:** Load and process documents.
2. **Chunking:** Split documents into smaller sections.
3. **Embedding:** Convert chunks into numerical vectors.
4. **Storage:** Store vectors and metadata.
5. **Retrieval:** Find relevant information for a query.
6. **Context Construction:** Select and organize retrieved results.
7. **Generation:** Generate an answer using the query and retrieved context.
8. **Evaluation:** Measure retrieval quality, faithfulness, correctness, latency, and cost.

### 7.2 Key Design Choices

- **Chunking:** Controls how information is divided.
- **Embeddings:** Represent semantic information numerically.
- **Retrieval Models:** Determine which information is selected.
- **Vector Databases:** Store and search vectors.
- **Reranking:** Reorders results by relevance.
- **Metadata Filtering:** Restricts results using attributes such as date or source.

### Key Takeaway

- RAG quality depends heavily on retrieving the right information, not merely on the LLM generating a good response.

## Chapter 8: Knowledge Graphs and Graph RAG

### 8.1 Knowledge Graphs

- A knowledge graph represents information through entities and explicit relationships.

**Example:**

`Company A → acquired → Company B`

### 8.2 Graph RAG

- Graph RAG retrieves relevant entities, relationships, and connected information from a knowledge graph to provide context to an LLM.

### Use Cases

- Multi-hop questions
- Relationship analysis
- Connecting information across documents
- Understanding complex networks of entities

### Trade-Offs

- Graph construction increases system complexity.
- Entity resolution can be challenging.
- Maintaining the graph adds operational cost.

## Chapter 9: Agentic RAG

- **Definition:** Agentic RAG uses an AI agent to dynamically control retrieval instead of relying entirely on a fixed retrieval pipeline.

### 9.1 Typical Process

1. Understand the question.
2. Plan a retrieval strategy.
3. Select a tool or information source.
4. Retrieve information.
5. Evaluate whether the results are sufficient.
6. Search again if necessary.
7. Synthesize the evidence into an answer.

### 9.2 Use Cases

- Complex research
- Multi-source investigation
- Tasks requiring iterative searches

### 9.3 Trade-Offs

- Additional latency
- Higher token costs
- Potential tool failures
- Increased risk of reasoning errors

### Decision Rule

- Use agentic retrieval when adaptive search decisions provide enough value to justify the added complexity.

## Chapter 10: Vectorless RAG

- **Definition:** Vectorless RAG retrieves information without depending on embedding-based vector similarity.

### 10.1 Retrieval Methods

- Document hierarchies
- Headings and tables of contents
- Metadata and document identifiers
- Keywords and structural relationships

**Example:**

`Document → Chapter → Section → Subsection`

### 10.2 Use Cases

- Structured documents
- Exact lookups
- Workloads where document structure provides reliable retrieval signals

### 10.3 Trade-Offs

- May struggle when relevant information is expressed using different wording.
- May struggle when information is distributed across unrelated sections.

### Key Takeaway

- Vectorless retrieval uses document structure or other non-vector methods instead of relying on embeddings.

## Chapter 11: Cache-Augmented Generation (CAG)

- **Definition:** CAG reduces repeated retrieval work by preparing knowledge context in advance and, in supported implementations, reusing the model's key-value (KV) cache.

### 11.1 Typical Process

1. Load a known knowledge collection.
2. Prepare the context in advance.
3. Prefill the model with the knowledge.
4. Reuse the prepared context or KV cache when supported.
5. Generate responses without repeating the entire retrieval process.

### 11.2 Use Cases

- Stable knowledge collections
- Repeated queries over the same information
- Workloads where retrieval latency is a bottleneck

### 11.3 Trade-Offs

- Large contexts consume memory.
- Knowledge updates require careful cache management.

### Important Note

- CAG does not permanently teach the model new knowledge.
- It makes prepared context or cached computation reusable.

## Chapter 12: Context Stuffing

- **Definition:** Context stuffing places the entire relevant document, or a large portion of it, directly into the LLM's context window instead of first retrieving selected chunks.

### 12.1 Typical Process

1. Load the document.
2. Convert it into a model-readable format.
3. Insert the content into the prompt.
4. Ask the LLM to analyze the supplied information.

### 12.2 Use Cases

- Short documents
- Small document collections
- Tasks requiring a broad view of the source
- Prototypes where a retrieval pipeline is unnecessary

### 12.3 Trade-Offs

- Higher input-token consumption
- Potentially higher latency and cost
- Context-window limitations
- Risk of important information being overlooked

### Decision Rule

- Use context stuffing when the relevant material fits comfortably within the context window and the cost is acceptable.

## Chapter 13: Comparison of External Knowledge Approaches

### 13.1 Traditional RAG

- **Core mechanism:** Retrieves relevant document chunks.
- **Best suited for:** Large knowledge bases.

### 13.2 Graph RAG

- **Core mechanism:** Traverses entities and relationships.
- **Best suited for:** Relationship-heavy questions.

### 13.3 Agentic RAG

- **Core mechanism:** Dynamically plans and executes retrieval.
- **Best suited for:** Complex research.

### 13.4 Vectorless RAG

- **Core mechanism:** Uses structure, metadata, or keywords.
- **Best suited for:** Structured documents.

### 13.5 Cache-Augmented Generation (CAG)

- **Core mechanism:** Reuses prepared knowledge context or cached computation.
- **Best suited for:** Repeated queries over stable knowledge.

### 13.6 Context Stuffing

- **Core mechanism:** Loads the full relevant source into the prompt.
- **Best suited for:** Small document collections.

### Key Takeaway

- These approaches are not mutually exclusive.
- A production system can combine retrieval, graph traversal, caching, and metadata filtering.

## Chapter 14: Choosing the Right Approach

### 14.1 Traditional RAG

- Choose when the knowledge base is large and only relevant subsets should enter the context.

### 14.2 Graph RAG

- Choose when relationships between entities are essential.

### 14.3 Agentic RAG

- Choose when retrieval requires dynamic decisions and multiple searches.

### 14.4 Vectorless RAG

- Choose when document structure or metadata supports reliable retrieval.

### 14.5 Cache-Augmented Generation (CAG)

- Choose when knowledge is stable and repeated queries make context reuse beneficial.

### 14.6 Context Stuffing

- Choose when the relevant source is small enough to include directly.

### 14.7 Evaluation Criteria

Evaluate each approach using the following criteria:

- **Answer Quality and Faithfulness:** Does the response accurately reflect the available evidence?
- **Retrieval Accuracy:** Does the system retrieve the relevant information?
- **Data Freshness:** Is the information sufficiently up to date?
- **Context-Window Usage:** How much context does the model need?
- **Latency:** How long does the system take to respond?
- **Inference and Infrastructure Cost:** What are the computational and operational costs?
- **Operational Complexity:** How difficult is the system to deploy, maintain, and debug?

## Final Chapter: Key Takeaways

- Context engineering is about selecting and organizing the right information, not simply adding more context.
- An LLM's output depends on the instructions, knowledge, tools, memory, state, and prompt available to it.
- RAG retrieves relevant external information to ground generation.
- Graph RAG uses relationships between entities.
- Agentic RAG adapts retrieval dynamically.
- Vectorless RAG uses non-vector retrieval mechanisms.
- CAG reduces repeated work through prepared context or reusable cached computation.
- Context stuffing supplies the full relevant source directly when practical.
- No single approach is best for every application.

### Core Principle

> Provide sufficient, relevant, and trustworthy context while balancing response quality, latency, cost, and system complexity.