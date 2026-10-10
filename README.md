# Agentic RAG with Typesense, Tavily and LangGraph

A beginner-friendly Agentic RAG (Retrieval-Augmented Generation) project. An LLM agent answers questions by choosing between two tools:

1. **Typesense retriever**: searches a vector store built from lecture transcript PDFs (CME 295, Transformers and Large Language Models).
2. **Tavily search**: searches the web for recent or out-of-scope information.

The agent decides on its own whether to search the lecture notes, search the web, or answer directly without any tool.

---

## Table of Contents

- [How It Works](#how-it-works)
- [Tech Stack](#tech-stack)
- [Notebook Walkthrough](#notebook-walkthrough)
- [Example Queries](#example-queries)
- [Future Improvements](#future-improvements)

---

## How It Works

The project has two stages.

### Stage 1: Build the knowledge base

1. PDFs are loaded from the `data/pdf` folder using `PyMuPDFLoader`.
2. Each page is split into smaller chunks (1000 characters, 200 overlap) using `RecursiveCharacterTextSplitter`.
3. Each chunk is converted into a vector using the `all-MiniLM-L6-v2` embedding model.
4. The vectors are stored in a Typesense collection named `RAG`.

### Stage 2: Run the agent

The agent is a LangGraph graph with two nodes, an agent node (the LLM) and a tools node (runs the retriever or Tavily).

```
START --> agent --(tool requested?)--> tools
            ^                            |
            |----------------------------|
            |
      (no tool needed)
            |
            v
           END
```

1. The user question goes to the agent node.
2. The LLM either answers directly or requests a tool call.
3. If a tool is requested, the tools node runs it and returns the result to the agent.
4. The loop repeats until the LLM produces a final answer.

---

## Tech Stack

| Component | Tool |
|---|---|
| Orchestration | LangChain, LangGraph |
| Vector store | Typesense (Typesense Cloud) |
| Embeddings | `all-MiniLM-L6-v2` via HuggingFace |
| PDF loading | PyMuPDF |
| Text splitting | RecursiveCharacterTextSplitter |
| Web search | Tavily |
| LLMs | Google Gemini (`llm1`), Groq `gpt-oss-120b` (`llm2`) |

---


## Notebook Walkthrough

| Section | What it does |
|---|---|
| Setting up TypeSense Vector Store | Loads PDFs, splits them into chunks, creates embeddings, uploads them to Typesense, and builds a retriever |
| Agentic RAG: LLM setup | Loads environment variables and initializes the Gemini and Groq models |
| Tools | Wraps the retriever as a tool (`lecture_notes_search`) and creates the Tavily web search tool |
| Agent node | Binds the tools to the LLM and defines the function that calls it with a system prompt |
| Graph | Builds the LangGraph flow with the agent node, tools node and conditional routing |
| Testing | Runs sample questions to check that the agent picks the right tool |

---

## Example Queries

| Question | Expected behavior |
|---|---|
| Why do we use masked multi-head attention in the decoder? | Calls `lecture_notes_search` and answers from the lecture PDFs |
| What are the latest AI model releases this week? | Calls `tavily_search` and answers from web results |
| What is 15 multiplied by 4? | Answers directly with no tool call |

---

## Future Improvements

- Add a document grader that checks whether retrieved chunks are relevant to the question.
- Add a query rewriting step when retrieval quality is low.
- Fall back to Tavily automatically when the retrieved chunks are not relevant.
- Add conversation memory so the agent can handle follow-up questions.
- Wrap the agent in a simple FastAPI or Streamlit interface.
