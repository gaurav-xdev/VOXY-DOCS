# 050 - Memory Runtime Specification

## Purpose

Define the Memory Runtime subsystem that orchestrates all memory operations, manages memory storage backends, and provides a unified interface for the four memory types: Working, Episodic, Semantic, and Procedural.

## Scope

- Memory subsystem orchestration
- Storage backend management
- Memory type coordination
- Consolidation scheduling
- Retrieval routing
- Memory lifecycle management

## Responsibilities

- Coordinate between memory types
- Manage SQLite and sqlite-vec backends
- Schedule consolidation tasks
- Route retrieval requests
- Enforce memory quotas
- Provide unified memory API

## Non-Goals

- Specific memory algorithms (handled by type-specific subsystems)
- UI for memory browsing
- Memory export/import

## Architecture Position

Memory Runtime sits above the four memory types and below the Cognitive Engine. It is the facade for all memory operations.

## Inputs

- Store requests from Cognitive Engine
- Retrieve requests from Context Assembly
- Consolidation triggers
- Configuration updates

## Outputs

- Stored memory confirmations
- Retrieved memories
- Consolidation status
- Memory metrics

## Interfaces

```rust
pub struct MemoryRuntime {
    pub fn store(entry: MemoryEntry) -> Result<MemoryId, MemoryError>;
    pub fn retrieve(query: MemoryQuery) -> Result<Vec<MemoryEntry>, MemoryError>;
    pub fn delete(id: MemoryId) -> Result<(), MemoryError>;
    pub fn consolidate() -> Result<ConsolidationReport, MemoryError>;
    pub fn get_stats() -> MemoryStats;
}
```

## Data Models

### MemoryEntry

| Field | Type | Description |
|-------|------|-------------|
| id | MemoryId | Unique identifier |
| type | MemoryType | Working, Episodic, Semantic, Procedural |
| content | String | Memory content |
| embedding | Option<Vec<f32>> | Vector embedding |
| metadata | Map | Additional metadata |
| created_at | DateTime | Creation time |
| accessed_at | DateTime | Last access time |
| access_count | u32 | Access frequency |

### MemoryQuery

| Field | Type | Description |
|-------|------|-------------|
| text | Option<String> | Text query |
| embedding | Option<Vec<f32>> | Vector query |
| types | Vec<MemoryType> | Memory types to search |
| filters | Vec<Filter> | Metadata filters |
| top_k | usize | Max results |
| recency_weight | f32 | Recency bias |

### MemoryStats

| Field | Type | Description |
|-------|------|-------------|
| total_entries | usize | Total memories |
| by_type | Map | Count per type |
| storage_size_mb | f64 | Database size |
| last_consolidation | DateTime | Last consolidation |

## State Management

- Memory state is persisted in SQLite
- Runtime state is in-memory
- Consolidation state is tracked

## Threading Model

Memory operations run on Tokio blocking pool. SQLite is accessed via `rusqlite` with connection pooling.

## IPC Requirements

- No IPC for memory operations
- Memory stats may be queried via IPC

## Event Definitions

| Event | Payload | Description |
|-------|---------|-------------|
| `memory.stored` | MemoryEntry | Memory stored |
| `memory.retrieved` | (query, count) | Memories retrieved |
| `memory.deleted` | MemoryId | Memory deleted |
| `memory.consolidated` | ConsolidationReport | Consolidation complete |
| `memory.quota_warning` | usage | Approaching quota |

## Error Handling

| Code | Description | Recovery |
|------|-------------|----------|
| `ME001` | Storage full | Alert, stop storing |
| `ME002` | Embedding failure | Store without embedding |
| `ME003` | Query timeout | Return partial results |
| `ME004` | Database locked | Retry with backoff |
| `ME005` | Corruption detected | Restore from backup |

## Retry Strategy

- Store: 3 retries
- Retrieve: 2 retries
- Consolidation: 1 retry

## Recovery Strategy

- Storage full: Archive old memories
- Database locked: Wait and retry
- Corruption: Restore from backup, alert

## Security Requirements

- Memory database encrypted at rest (SQLite encryption)
- Access controlled by capability tokens
- No external sharing without consent

## Configuration

```json
{
  "memory_runtime": {
    "database_path": "%LOCALAPPDATA%/VOXY/memory.db",
    "max_size_mb": 1024,
    "consolidation_interval_hours": 24,
    "embedding_model": "local",
    "embedding_dimensions": 384,
    "enable_encryption": true,
    "backup_interval_days": 7
  }
}
```

## Performance Targets

| Metric | Target |
|--------|--------|
| Store latency | < 50ms |
| Retrieve latency | < 100ms |
| Consolidation | < 5 minutes |
| Database size | < 1GB |

## Resource Limits

| Resource | Limit |
|----------|-------|
| Database size | 1GB |
| Max entries | 1,000,000 |
| Embedding dimensions | 384 |
| Concurrent operations | 20 |

## Benchmarks

- `bench_store`: Storage latency
- `bench_retrieve`: Retrieval latency
- `bench_consolidation`: Consolidation speed

## Testing Requirements

- Storage/retrieval tests
- Consolidation tests
- Quota enforcement tests
- Encryption tests

## Logging Requirements

- Store/retrieve at DEBUG level
- Consolidation at INFO level
- Errors at ERROR level
- Quota warnings at WARN level

## Telemetry

- Operations per second
- Storage size
- Query latency
- Consolidation duration

## Cross References

- [051_WORKING_MEMORY_SPEC.md](051_WORKING_MEMORY_SPEC.md)
- [052_EPISODIC_MEMORY_SPEC.md](052_EPISODIC_MEMORY_SPEC.md)
- [053_SEMANTIC_MEMORY_SPEC.md](053_SEMANTIC_MEMORY_SPEC.md)
- [054_PROCEDURAL_MEMORY_SPEC.md](054_PROCEDURAL_MEMORY_SPEC.md)
- [055_MEMORY_CONSOLIDATION_SPEC.md](055_MEMORY_CONSOLIDATION_SPEC.md)
- [056_MEMORY_RETRIEVAL_SPEC.md](056_MEMORY_RETRIEVAL_SPEC.md)

## References

- SQLite
- sqlite-vec
- Vector Database Design
- Memory Systems in AI
