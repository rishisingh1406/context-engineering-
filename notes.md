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

## Key Takeaway

Context engineering is not just about adding more information. It is about selecting, organizing, updating, and delivering the right information to the LLM at the right time.