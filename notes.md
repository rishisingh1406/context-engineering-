# Context Engineering — Quick Notes

## Chapter 1: Definition
- Structures and manages LLM inputs.
- Goal: Balance relevance, token usage, and cost.

## Chapter 2: LLM System Architecture
- **LLM Calls:** Direct input-output.
- **Workflows:** Predefined execution steps.
- **Agents:** Dynamically select actions.
- **Agentic Systems:** Coordinate multiple agents or components.

## Chapter 3: LLM Limitations
- Hallucinations and stale data.
- Limited context windows.
- Information integrity issues.
- Safety risks and bias.

## Chapter 4: Benefits
- Reduces hallucinations.
- Improves consistency and reliability.
- Controls token usage and cost.
- Manages system complexity.

## Chapter 5: Six Sources of Context
- **Instructions:** Rules and constraints.
- **External Knowledge:** Documents, databases, APIs.
- **Tools:** Functions and outputs.
- **Memory:** Retained information.
- **State:** Current task or workflow status.
- **User Prompts:** Intent and requirements.

## Chapter 6: External Knowledge
- Supplies information missing from or newer than model training data.
- Sources: Documents, databases, APIs, websites, and knowledge graphs.
- Goal: Ground responses in relevant, verifiable information.

## Chapter 7: Retrieval-Augmented Generation (RAG)
- Retrieves external information to ground LLM responses.
- **Pipeline:** Indexing → Chunking → Embedding → Storage → Retrieval → Context Construction → Generation → Evaluation.
- **Key design choices:** Chunking, embeddings, retrieval, vector databases, reranking, and metadata filtering.
- Quality depends heavily on retrieving relevant information.

## Chapter 8: Knowledge Graphs and Graph RAG
- **Knowledge Graph:** Represents entities and relationships.
- **Graph RAG:** Retrieves connected entities and relationships.
- **Use cases:** Multi-hop questions and relationship analysis.
- **Trade-off:** Higher construction and maintenance complexity.

## Chapter 9: Agentic RAG
- Agents dynamically plan and control retrieval.
- **Process:** Understand → Plan → Retrieve → Evaluate → Search Again if Needed → Synthesize.
- **Use cases:** Complex, multi-source research.
- **Trade-off:** Higher latency, cost, and potential failure risk.

## Chapter 10: Vectorless RAG
- Retrieves information without embedding-based vector similarity.
- Uses document structure, headings, metadata, and keywords.
- **Best for:** Structured documents and exact lookups.
- **Limitation:** May miss semantically related information expressed differently.

## Chapter 11: Cache-Augmented Generation (CAG)
- Prepares knowledge context for reuse, potentially using the KV cache.
- **Best for:** Stable knowledge and repeated queries.
- **Trade-off:** Memory usage and cache-update complexity.
- Does not permanently teach the model new knowledge.

## Chapter 12: Context Stuffing
- Places the relevant document directly into the context window.
- **Best for:** Short documents and broad document analysis.
- **Trade-off:** Higher token usage, cost, latency, and context-window pressure.

## Chapter 13: Comparing Knowledge Approaches
- **Traditional RAG:** Retrieves relevant chunks.
- **Graph RAG:** Uses entity relationships.
- **Agentic RAG:** Dynamically plans retrieval.
- **Vectorless RAG:** Uses structure and metadata.
- **CAG:** Reuses prepared context or cached computation.
- **Context Stuffing:** Supplies the full relevant source.
- Approaches can be combined in one system.

## Chapter 14: Choosing the Right Approach
- **Traditional RAG:** Large knowledge bases.
- **Graph RAG:** Relationship-heavy questions.
- **Agentic RAG:** Complex, adaptive research.
- **Vectorless RAG:** Structured documents.
- **CAG:** Repeated queries over stable knowledge.
- **Context Stuffing:** Small sources that fit comfortably in context.

### Evaluation Criteria
- Answer quality and faithfulness.
- Retrieval accuracy and data freshness.
- Context-window usage.
- Latency and cost.
- Operational complexity.

## Final Chapter: Key Takeaways
- Context engineering selects and organizes useful information.
- Context includes instructions, knowledge, tools, memory, state, and prompts.
- Each retrieval approach has different strengths and trade-offs.
- No single approach fits every application.

### Core Principle
> Provide relevant, trustworthy context while balancing quality, latency, cost, and complexity.