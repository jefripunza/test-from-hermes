---
name: prompt-engineering
description: "Prompt engineering expert: prompt design, agent workflows, task decomposition, context management, tool usage, reasoning structures. Reliable AI execution, predictable outputs."
metadata:
  hermes:
    tags: [prompt-engineering, llm, agents, reasoning, context-management, tool-use]
---

# Prompt Engineering — Reliable AI Execution

Design prompts & agent workflows for reliable, predictable AI execution.

## Expertise

- **Prompt Design** — system prompts, role definitions, output formatting
- **Agent Workflows** — tool-calling loops, multi-turn, delegation
- **Task Decomposition** — break complex tasks into subtasks
- **Context Management** — token budgeting, compression, progressive disclosure
- **Tool Usage** — schema design, parameter patterns, error handling
- **Reasoning Structures** — chain-of-thought, tree-of-thought, ReAct

## Prompt Design Principles

### System Prompt Structure

```text
Role: Who the AI is (expert, assistant, reviewer)
Context: Background, constraints, environment
Task: What to do, what not to do
Format: Output structure, tone, style
Examples: Few-shot (2-3) for complex tasks
```

### Anti-Patterns

- **Ambiguous roles** — "you are a helpful assistant" vs "you are a senior Go backend engineer with 10 years experience"
- **No guardrails** — define what NOT to do explicitly
- **Information overload** — too much context crushes signal
- **Conflicting instructions** — earlier instruction contradicts later one
- **No output structure** — freeform = unreliable parsing

### Do's

- Specific, measurable instructions
- Format examples for structured output
- Define priority when instructions conflict
- Use delimiters (```, ---, ===) for clarity
- Request chain-of-thought before final answer

## Task Decomposition

For complex tasks, decompose:

```text
1. Understand — gather context, ask clarifying questions
2. Plan — break into subtasks, order dependencies
3. Execute — do each subtask, verify intermediate results
4. Review — check correctness, completeness, quality
5. Refine — fix issues, optimize
```

Tools per phase:
- Understand: `read_file`, `web_search`, `clarify`
- Plan: `todo`, `skill_view("plan")`
- Execute: `terminal`, `write_file`, `patch`
- Review: `skill_view("senior-review")`

## Tool Usage Patterns

### Schema Design

```json
{
  "name": "search_users",
  "description": "Search users by email or name. Returns up to 10 results.",
  "parameters": {
    "type": "object",
    "properties": {
      "query": { "type": "string", "description": "Email or name to search" },
      "limit": { "type": "integer", "default": 10 }
    },
    "required": ["query"]
  }
}
```

### Best Practices

- **Descriptive names** — `search_users` not `search1`
- **Clear descriptions** — include when to use, what it returns
- **Required params** — minimal set, sensible defaults
- **Return JSON** — structured, parseable, includes errors
- **Error handling** — return error messages, don't crash

## Reasoning Structures

### Chain-of-Thought (CoT)

```
Step 1: Understand the problem
Step 2: Break into sub-problems
Step 3: Solve each sub-problem
Step 4: Combine results
Therefore: ...
```

### ReAct (Reasoning + Acting)

```
Thought: I need to find the user's email
Action: search_users(query="John Doe")
Observation: Found user jdoe@example.com
Thought: Now I can send the notification
Action: send_email(to="jdoe@example.com", subject="...")
Observation: Email sent successfully
Thought: Task complete
```

### Structured Output (JSON Mode)

```json
{
  "analysis": "Step-by-step reasoning",
  "conclusion": "Final answer",
  "confidence": 0.95,
  "evidence": ["fact1", "fact2"]
}
```

## Context Management

| Strategy | When | How |
|----------|------|-----|
| Progressive disclosure | Large context | Load skill → reference → full file only if needed |
| Compression | Token budget full | Summarize past turns, keep decisions |
| Summarization | Long conversation | Compact the key points, drop details |
| Prioritization | Mixed content | Important facts first, reference later |

## Goal

- Reliable AI execution
- Predictable outputs
- Structured, parseable results
