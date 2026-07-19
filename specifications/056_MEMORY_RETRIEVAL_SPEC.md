# 056 - Memory Retrieval Specification

## Purpose

Define the Memory Retrieval subsystem that searches across all memory types, ranks results by relevance, and returns the most pertinent information for context assembly.

## Scope

- Cross-memory-type search
- Vector similarity search
- Keyword and metadata filtering
- Result ranking and deduplication
- Recency and importance weighting
- Query expansion

## Responsibilities

- Search across all memory types
- Rank results by relevance
- Deduplicate results
- Apply recency and importance weights
- Expand queries for better recall
- Return formatted results

## Non-Goals

- Memory storage (individual memory subsystems)
- Context assembly (Context Assembly)
- Embedding generation (Embedding subsystem)

## Architecture Position

Memory Retrieval sits between Context Assembly and the four memory types. It is the search engine for VOXY's memory.

## Inputs

- Query text or embeddings
- Memory type filters
- Recency/importance preferences
- Result count limits

## Outputs

- Ranked memory results
- Relevance scores
- Source attributions
- Search metrics

## Interfaces

```rust
pub struct MemoryRetrieval {
    pub fn search(query: &str, config: SearchConfig) -> Result<SearchResult, RetrievalError>;
    pub fn search_by_embedding(embedding: &[f32], config: SearchConfig) -> Result<SearchResult, RetrievalError>;
    pub fn hybrid_search(query: &str, embedding: &[f32], config: SearchConfig) -> Result<SearchResult, RetrievalError>;
    pub fn get_recent(limit: usize, types: Vec<MemoryType>) -> Vec<MemoryEntry>;
}
```

## Data Models

### SearchConfig

| Field | Type | Description |
|-------|------|-------------|
| types | Vec<MemoryType> | Memory types to search |
| top_k | usize | Max results |
| recency_weight | f32 | Recency bias (0-1) |
| importance_weight | f32 | Importance bias (0-1) |
| similarity_threshold | f32 | Minimum similarity |
| time_range | Option<TimeRange> | Temporal filter |

### SearchResult

| Field | Type | Description |
|-------|------|-------------|
| results | Vec<RankedEntry> | Ranked memories |
| total_found | usize | Total matches |
| query_time_ms | u64 | Search latency |

### RankedEntry

| Field | Type | Description |
|-------|------|-------------|
| entry | MemoryEntry | Memory entry |
| relevance_score | f32 | Combined relevance |
| vector_score | f32 | Vector similarity |
| recency_score | f32 | Recency score |
| importance_score | f32 | Importance score |

## State Management

- No persistent retrieval state
- Query cache for repeated queries

## Threading Model

Searches run on blocking pool. Multiple memory types searched in parallel.

## IPC Requirements

- No IPC for retrieval

## Event Definitions

| Event | Payload | Description |
|-------|---------|-------------|
| `retrieval.search` | SearchConfig | Search performed |
| `retrieval.results` | SearchResult | Results returned |
| `retrieval.cache_hit` | query | Cache hit |

## Error Handling

| Code | Description | Recovery |
|------|-------------|----------|
| `RE001` | Search timeout | Return partial results |
| `RE002` | Index error | Fallback to keyword search |
| `RE003` | No results | Return empty |

## Retry Strategy

- Search: 1 retry
- Fallback: Immediate

## Recovery Strategy

- Timeout: Return partial
- Index error: Keyword fallback

## Security Requirements

- Results filtered by capability tokens
- No unauthorized memory access

## Configuration

```json
{
  "memory_retrieval": {
    "default_top_k": 10,
    "max_top_k": 100,
    "cache_enabled": true,
    "cache_ttl_seconds": 300,
    "search_timeout_ms": 5000,
    "enable_query_expansion": true,
    "vector_search_enabled": true
  }
}
```

## Performance Targets

| Metric | Target |
|--------|--------|
| Search latency | < 100ms |
| Cache hit rate | > 50% |
| Result relevance | > 0.8 |

## Resource Limits

| Resource | Limit |
|----------|-------|
| Cache size | 1000 entries |
| Max results | 100 |
| Query length | 1000 chars |

## Benchmarks

- `bench_search`: Search latency
- `bench_hybrid`: Hybrid search quality
- `bench_ranking`: Ranking accuracy

## Testing Requirements

- Search accuracy tests
- Ranking quality tests
- Cache tests
- Timeout tests

## Logging Requirements

- Searches at TRACE level
- Errors at ERROR level

## Telemetry

- Search frequency
- Average latency
- Cache hit rate
- Result count

## Cross References

- [050_MEMORY_RUNTIME_SPEC.md](050_MEMORY_RUNTIME_SPEC.md)
- [040_CONTEXT_ASSEMBLY_SPEC.md](040_CONTEXT_ASSEMBLY_SPEC.md)
- [101_VECTOR_DATABASE_SPEC.md](101_VECTOR_DATABASE_SPEC.md)
- [102_EMBEDDING_SPEC.md](102_EMBEDDING_SPEC.md)

## References

- Information Retrieval
- Vector Search
- Hybrid Search (BM25 + Vector)
- Recency Ranking
