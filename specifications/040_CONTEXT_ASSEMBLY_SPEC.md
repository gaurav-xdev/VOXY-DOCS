# 040 - Context Assembly Specification

## Purpose

Define the Context Assembly subsystem that constructs the optimal context window for LLM inference by aggregating working memory, retrieved episodic/semantic memories, system prompts, and current session state.

## Scope

- Context window construction
- Token budget management
- Memory prioritization and ranking
- System prompt injection
- Conversation history management
- Context compression and summarization

## Responsibilities

- Assemble context from multiple sources
- Enforce token budget limits
- Prioritize relevant memories
- Manage conversation history
- Compress context when over budget
- Track context versions for debugging

## Non-Goals

- LLM inference (Model Router responsibility)
- Memory storage (Memory Subsystem responsibility)
- Tool execution (Executor responsibility)

## Architecture Position

Context Assembly sits between Memory Retrieval and Model Router, preparing the input for LLM inference.

## Inputs

- Current user query
- Working memory state
- Retrieved episodic memories
- Retrieved semantic memories
- System prompts and instructions
- Tool schemas
- Conversation history

## Outputs

- Assembled context window
- Token usage report
- Context attribution (source tracking)
- Compression metrics

## Interfaces

```rust
pub struct ContextAssembly {
    pub fn assemble(request: AssemblyRequest) -> Result<ContextWindow, AssemblyError>;
    pub fn get_token_count(text: &str) -> usize;
    pub fn compress(context: &ContextWindow, target_tokens: usize) -> Result<ContextWindow, AssemblyError>;
    pub fn get_attribution() -> Vec<Attribution>;
}
```

## Data Models

### ContextWindow

| Field | Type | Description |
|-------|------|-------------|
| messages | Vec<Message> | Ordered messages for LLM |
| token_count | usize | Total tokens |
| sources | Vec<Source> | Attribution sources |
| version | u32 | Assembly version |

### Message

| Field | Type | Description |
|-------|------|-------------|
| role | Role | System, User, Assistant, Tool |
| content | String | Message content |
| name | Option<String> | Tool name |
| tool_calls | Option<Vec<ToolCall>> | Tool invocations |

### AssemblyRequest

| Field | Type | Description |
|-------|------|-------------|
| query | String | Current user input |
| max_tokens | usize | Token budget |
| include_memories | bool | Include retrieved memories |
| include_tools | bool | Include tool schemas |
| priority | Priority | Assembly priority |

## State Management

- Context windows are immutable after assembly
- History is maintained in Working Memory
- Assembly versions are tracked for debugging

## Threading Model

Context assembly runs on Tokio blocking pool. Token counting is CPU-bound.

## IPC Requirements

- No IPC for context assembly
- Context may be logged for debugging

## Event Definitions

| Event | Payload | Description |
|-------|---------|-------------|
| `context.assembled` | ContextWindow | Context assembled |
| `context.compressed` | (old, new) | Context compressed |
| `context.token_budget_exceeded` | usage | Budget exceeded |

## Error Handling

| Code | Description | Recovery |
|------|-------------|----------|
| `CA001` | Token budget exceeded | Compress, truncate |
| `CA002` | Assembly timeout | Return partial context |
| `CA003` | Invalid source | Skip source, log warning |

## Retry Strategy

- Assembly: 1 retry with reduced budget
- Compression: 2 retries

## Recovery Strategy

- Budget exceeded: Compress history, drop low-priority memories
- Timeout: Return best-effort context

## Security Requirements

- System prompts are immutable (template-based)
- User content is sanitized
- Memory content is filtered by permission

## Configuration

```json
{
  "context_assembly": {
    "max_tokens": 8192,
    "system_prompt_tokens": 500,
    "history_tokens": 3000,
    "memory_tokens": 3000,
    "tool_tokens": 1000,
    "reserve_tokens": 692,
    "compression_enabled": true,
    "compression_threshold": 0.9
  }
}
```

## Performance Targets

| Metric | Target |
|--------|--------|
| Assembly latency | < 50ms |
| Token counting | < 10ms per 1K tokens |
| Compression | < 100ms |

## Resource Limits

| Resource | Limit |
|----------|-------|
| Max context size | 128K tokens |
| History entries | 1000 |
| Memory entries per query | 20 |

## Benchmarks

- `bench_assembly`: Full assembly latency
- `bench_compression`: Compression quality/speed
- `bench_token_count`: Token counting speed

## Testing Requirements

- Token budget enforcement tests
- Compression quality tests
- Attribution accuracy tests
- Priority ordering tests

## Logging Requirements

- Assembly events at DEBUG level
- Budget exceeded at WARN level
- Compression at INFO level

## Telemetry

- Assembly latency
- Token usage by source
- Compression frequency
- Budget exceedance rate

## Cross References

- [041_MODEL_ROUTER_SPEC.md](041_MODEL_ROUTER_SPEC.md)
- [051_WORKING_MEMORY_SPEC.md](051_WORKING_MEMORY_SPEC.md)
- [056_MEMORY_RETRIEVAL_SPEC.md](056_MEMORY_RETRIEVAL_SPEC.md)

## References

- OpenAI Context Window Management
- Anthropic Context Assembly
- LangChain Context Compression
