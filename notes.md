# Context Engineering

## 1. Definition

Context engineering is the discipline of structuring and managing all inputs that shape how an LLM interprets and responds to a task.

It balances relevance, brevity, and cost to ensure the most useful information fits within the model's limited context window.

## 2. Levels of LLM System Architecture

1. **LLM Calls:** A direct input-output interaction with an LLM.
2. **AI Workflows:** Predefined steps control how the LLM and other components execute tasks.
3. **AI Agents:** An LLM selects actions and iterates through an agent loop.
4. **Agentic Systems:** One or more agents coordinate within a broader runtime environment.

## 3. Key Limitations of LLMs

- **Hallucinations:** Generating incorrect or unsupported information.
- **Stale Data:** Relying on outdated information.
- **Limited Context Window:** Restrictions on how much information the model can process at once.
- **Information Integrity:** Difficulty ensuring that inputs and outputs remain accurate and trustworthy.
- **Safety:** Potentially producing harmful or policy-violating outputs.
- **Bias:** Reflecting biases present in training data or context.

## 4. Benefits of Context Engineering

- Reduces hallucinations by providing relevant, grounded information.
- Improves response consistency across similar tasks.
- Controls token usage and inference costs.
- Helps manage context as systems grow in complexity.
- Improves the reliability of production AI systems.

## 5. Six Sources of Context

1. **Instructions:** High-level guidelines, constraints, rules, and behavioral requirements.
2. **External Knowledge:** Retrieved information from documents, databases, APIs, or other sources.
3. **Tools:** Available function definitions, commands, and tool outputs.
4. **Memory:** Relevant information retained across interactions or sessions.
5. **State:** The current status of a task, workflow, or environment.
6. **User Prompts:** The user's immediate request, intent, and requirements.

## 6. External Knowledge

External knowledge grounds LLM outputs in factual, up-to-date information that may be unavailable in the model's training data.

Instead of relying entirely on learned parameters, an LLM system can retrieve information from external sources and inject relevant content into the model's context window.

**Common sources:**
- Documents and PDFs
- Databases and knowledge bases
- APIs and live data sources
- Websites and search engines
- Knowledge graphs

External knowledge is particularly useful when an application needs domain-specific, private, frequently updated, or verifiable information.

## 7. Retrieval-Augmented Generation (RAG)

RAG combines information retrieval with LLM generation.

Instead of asking the LLM to answer using only its internal knowledge, the system retrieves relevant external information and provides it as context for generating a response.

### RAG Pipeline

1. **Indexing:** Load, clean, and process source documents.
2. **Chunking:** Divide documents into smaller sections.
3. **Embedding:** Convert chunks into numerical vector representations.
4. **Storage:** Store vectors and metadata in a vector database.
5. **Retrieval:** Find relevant chunks for a user's query.
6. **Context Construction:** Select and organize the retrieved information.
7. **Generation:** Supply the context and query to the LLM.
8. **Evaluation:** Measure retrieval quality, answer correctness, faithfulness, latency, and cost.

### Key Design Choices

- **Chunking:** Determines how documents are divided and how much context each chunk preserves.
- **Embeddings:** Represent semantic information numerically for similarity-based retrieval.
- **Retrieval Models:** Determine which information is selected for a query.
- **Vector Databases:** Store and search vector representations.
- **Reranking:** Reorders retrieved results to prioritize the most relevant information.
- **Metadata Filtering:** Restricts retrieval using attributes such as source, date, or document type.

These choices strongly influence retrieval quality, answer accuracy, latency, and cost.

## 8. Knowledge Graphs and Graph RAG

### Knowledge Graphs

A knowledge graph represents information as entities and explicit relationships.

For example:

`Company A → acquired → Company B`

`Company A → operates_in → India`

Knowledge graphs preserve relationships between entities rather than representing information only as independent text chunks.

### Graph RAG

Graph RAG uses a knowledge graph to retrieve relevant entities, relationships, and connected information for an LLM.

Instead of relying only on vector similarity, the system can traverse graph relationships to gather context.

**Useful for:**
- Multi-hop questions
- Entity and relationship analysis
- Connecting information across documents
- Understanding complex relationships in large datasets

**Trade-off:** Graph construction, entity resolution, maintenance, and traversal can increase system complexity and cost.

## 9. Agentic RAG

Agentic RAG uses an AI agent to control the retrieval process dynamically.

Unlike a conventional RAG pipeline with a mostly predefined retrieval sequence, an agent can decide which search strategy to use, whether more information is needed, and when to stop searching.

### Typical Process

1. Understand the user's question.
2. Plan a retrieval strategy.
3. Select a tool or information source.
4. Retrieve information.
5. Evaluate whether the results are sufficient.
6. Perform additional searches if necessary.
7. Synthesize the evidence into a response.

**Useful for:**
- Complex research tasks
- Questions requiring multiple information sources
- Iterative investigation
- Tasks where the next retrieval action depends on previous results

**Trade-off:** Agentic RAG can improve flexibility, but it may also increase latency, token usage, tool errors, and the risk of unreliable reasoning.

Use it when dynamic retrieval decisions provide enough value to justify the additional complexity.

## 10. Vectorless RAG

Vectorless RAG retrieves information without relying on embedding-based vector similarity.

Instead, it uses document structure, metadata, hierarchies, keywords, or other non-vector retrieval mechanisms to identify relevant information.

For example, a system may navigate:

`Document → Chapter → Section → Subsection`

It can use headings, document identifiers, tables of contents, or structural relationships to locate the required content.

**Useful for:**
- Well-structured documents
- Documents with meaningful headings and hierarchies
- Exact section or metadata-based retrieval
- Workloads where embedding generation and vector search are unnecessary

**Trade-off:** It may perform poorly when relevant information is expressed using different wording or appears across unrelated sections.

Vectorless RAG is not one standardized architecture. It is a broad description of retrieval approaches that do not depend on vector embeddings.

## 11. Cache-Augmented Generation (CAG)

Cache-Augmented Generation (CAG) is an approach that reduces repeated retrieval work by preloading relevant knowledge into the model's available context or caching reusable model computation.

In a common CAG design, the system prepares the knowledge context in advance and may reuse the model's key-value (KV) cache across requests.

### Typical Process

1. Load a known knowledge collection.
2. Prepare the context before the user asks a question.
3. Prefill the model with the knowledge.
4. Reuse the prepared context or KV cache when supported.
5. Generate responses without repeating the entire retrieval process.

**Useful for:**
- Relatively small, stable knowledge collections
- Repeated queries over the same information
- Workloads where retrieval latency is a bottleneck

**Trade-off:** Large contexts consume memory and may increase prefill costs. Updating knowledge and managing cache validity can also be challenging.

**Important distinction:** CAG is not simply permanent storage of knowledge inside an LLM. Its effectiveness depends on the model's context limits, cache implementation, and knowledge-update strategy.

## 12. Context Stuffing

Context stuffing means placing the full relevant document or a large portion of the source material directly into the LLM's context window rather than retrieving selected chunks first.

### Typical Process

1. Load the source document.
2. Convert it into text or another supported representation.
3. Insert the content into the model's prompt.
4. Ask the LLM to analyze the provided information.

**Useful for:**
- Short documents
- Small, bounded document collections
- Tasks requiring a broad view of the supplied material
- Prototypes where building a retrieval pipeline is unnecessary

**Trade-off:**
- Consumes more input tokens.
- Can increase latency and inference costs.
- May exceed the context window.
- Can make important details harder for the model to use effectively.

Context stuffing does not eliminate the need for retrieval in every application. It is a practical alternative when the entire relevant source fits within the context window and the cost is acceptable.

## 13. Comparing External Knowledge Approaches

| Approach | How it obtains context | Best suited for |
|---|---|---|
| Traditional RAG | Retrieves relevant document chunks | Large or frequently updated collections |
| Graph RAG | Traverses entities and relationships | Relationship-heavy and multi-hop questions |
| Agentic RAG | Dynamically plans and executes retrieval | Complex, iterative research |
| Vectorless RAG | Uses structure, metadata, keywords, or hierarchies | Structured documents and exact lookups |
| CAG | Preloads or reuses cached knowledge context | Repeated queries over stable knowledge |
| Context Stuffing | Loads the full relevant source into the prompt | Small documents and bounded datasets |

These approaches are not mutually exclusive. A production system can combine them—for example, using metadata filtering with vector retrieval, graph traversal for relationship questions, and caching for repeated queries.

## 14. Choosing the Right Approach

- **Use traditional RAG** when the knowledge base is large and retrieving relevant subsets is necessary.
- **Use Graph RAG** when relationships between entities are central to answering questions.
- **Use Agentic RAG** when retrieval requires multiple adaptive decisions or searches.
- **Use Vectorless RAG** when document structure or exact metadata can locate information reliably.
- **Use CAG** when the same stable knowledge is reused frequently and caching is practical.
- **Use Context Stuffing** when the relevant material is small enough to fit comfortably within the context window.

The best architecture depends on retrieval quality, data freshness, context size, latency, cost, and operational complexity—not on which technique sounds most advanced.

## Key Takeaway

Context engineering is not just about adding more information. It is about selecting, organizing, updating, and delivering the right information to the LLM at the right time.

External knowledge techniques provide different ways to make information available to a model. Traditional RAG retrieves selected information, Graph RAG uses relationships, Agentic RAG adapts retrieval dynamically, Vectorless RAG relies on non-vector retrieval mechanisms, CAG reuses prepared knowledge context, and Context Stuffing supplies the full relevant source directly.

**The goal is to provide sufficient, relevant, trustworthy context while controlling latency, cost, and complexity.**