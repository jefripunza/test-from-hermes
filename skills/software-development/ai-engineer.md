---
name: ai-engineer
description: "AI engineer: OpenAI, Anthropic, RAG, embeddings, vector databases, agent architecture, tool calling, prompt engineering. Production-grade AI applications."
metadata:
  hermes:
    tags: [ai, llm, openai, anthropic, rag, embeddings, vector-database, agents]
---

# AI Engineer — Production AI Applications

Build production-grade AI applications. RAG, agents, embeddings, tool calling.

## Expertise

- **OpenAI** — GPT-4o, assistants API, structured outputs, vision
- **Anthropic** — Claude Opus/Sonnet, tool use, extended thinking
- **RAG** — retrieval augmented generation, chunking strategies, reranking
- **Embeddings** — text-embedding models, semantic search, hybrid search
- **Vector Databases** — pgvector, Pinecone, Qdrant, Weaviate, Milvus
- **Agent Architecture** — tool-calling loops, multi-agent, supervisor, orchestrator
- **Tool Calling** — function calling, structured output, parallel tool use
- **Prompt Engineering** — system prompts, few-shot, chain-of-thought

## RAG Architecture

```
Query
  → Embedding (user query)
  → Vector Search (top-k similar chunks)
  → Reranking (cross-encoder)
  → Context Assembly (chunks + query)
  → LLM Generation (grounded response)
```

### Chunking Strategies

| Strategy | Best For | Chunk Size |
|----------|----------|------------|
| Fixed size | General | 512-1024 tokens |
| Semantic | Coherent sections | Paragraph-level |
| Recursive | Code/doc | By delimiter (\n\n, ###) |
| Agentic | Complex docs | LLM-determine boundaries |

### Hybrid Search

```go
// pgvector hybrid search (semantic + keyword)
SELECT id, content,
  1 - (embedding <=> $1) AS semantic_score,
  ts_rank(to_tsvector('english', content), plainto_tsquery('english', $2)) AS keyword_score
FROM documents
WHERE 1 - (embedding <=> $1) > 0.7
   OR to_tsvector('english', content) @@ plainto_tsquery('english', $2)
ORDER BY (semantic_score * 0.7 + keyword_score * 0.3) DESC
LIMIT 20;
```

## Agent Architecture

```text
Single Agent:
  One loop: think → tool → observe → think → respond

Multi-Agent:
  Orchestrator: decompose task, assign sub-agents, merge results
  Worker: focused tool use, returns structured output
  Supervisor: quality check, re-delegate if needed

Patterns:
  - Supervisor (agent delegates to workers)
  - Router (classify input → route to specialized agent)
  - Tool-calling loop (standard function calling)
  - Reflection (agent critiques own output)
```

## Tool Calling Pattern

```go
type Tool struct {
    Name        string
    Description string
    Parameters  map[string]any
    Execute     func(args map[string]any) (string, error)
}

// Loop
for i := 0; i < maxIterations; i++ {
    response := llm.Chat(messages, tools)
    if response.IsText() { return response.Text }
    for _, toolCall := range response.ToolCalls {
        result := executeTool(toolCall)
        messages = append(messages, toolCall, result)
    }
}
```

## Production Considerations

- **Latency** — streaming, caching, batching
- **Cost** — token tracking, model selection (cheaper model first)
- **Reliability** — retry with backoff, fallback model, timeout
- **Observability** — trace LLM calls, log tokens, monitor latency p50/p95/p99
- **Safety** — input/output guardrails, PII redaction, content filters
- **Evaluation** — ground-truth eval, LLM-as-judge, A/B testing

## Goal

- Production-grade AI applications
- Reliable, observable, cost-effective
- **Production-ready architecture only**
