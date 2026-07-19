# 101 - Vector Database Specification

## Purpose

Define the vector database subsystem that enables semantic search across memories using sqlite-vec for efficient vector similarity queries.

## Scope

- Vector storage and indexing
- Similarity search
- Index management
- Vector normalization
- Distance metrics
- Performance optimization

## Responsibilities

- Store vector embeddings
- Support similarity search
- Manage vector indexes
- Normalize vectors
- Optimize query performance
- Handle vector dimensionality

## Non-Goals

- Embedding generation (Embedding subsystem)
- Full-text search (SQLite FTS)
- Distributed vector storage

## Architecture Position

Vector Database extends SQLite with vector capabilities via sqlite-vec.

## Data Models

### VectorIndex

| Field | Type | Description |
|-------|------|-------------|
| name | String | Index name |
| dimensions | usize | Vector dimensions |
| metric | DistanceMetric | Distance metric |
| entries | usize | Entry count |

### DistanceMetric

| Variant | Description |
|---------|-------------|
| `Cosine` | Cosine similarity |
| `Euclidean` | L2 distance |
| `DotProduct` | Dot product |

## Interfaces

```rust
pub struct VectorDatabase {
    pub fn create_index(name: &str, dimensions: usize, metric: DistanceMetric) -> Result<(), VectorError>;
    pub fn insert(index: &str, id: &str, vector: &[f32]) -> Result<(), VectorError>;
    pub fn search(index: &str, query: &[f32], top_k: usize) -> Result<Vec<SearchResult>, VectorError>;
    pub fn delete(index: &str, id: &str) -> Result<(), VectorError>;
    pub fn get_index_stats(index: &str) -> VectorIndex;
}
```

## State Management

- Indexes stored in SQLite
- No separate persistent state

## Threading Model

Vector operations run on blocking pool.

## IPC Requirements

- No IPC

## Event Definitions

| Event | Payload | Description |
|-------|---------|-------------|
| `vector.index_created` | name | Index created |
| `vector.inserted` | (index, id) | Vector inserted |
| `vector.searched` | (index, results) | Search performed |

## Error Handling

| Code | Description | Recovery |
|------|-------------|----------|
| `VD001` | Dimension mismatch | Reject |
| `VD002` | Index not found | Create |
| `VD003` | Search timeout | Return partial |

## Configuration

```json
{
  "vector_database": {
    "default_metric": "cosine",
    "default_dimensions": 384,
    "max_indexes": 10,
    "max_vectors_per_index": 1000000,
    "search_timeout_ms": 5000
  }
}
```

## Performance Targets

| Metric | Target |
|--------|--------|
| Insert latency | < 10ms |
| Search latency | < 100ms |
| Index build time | < 1 minute |

## Resource Limits

| Resource | Limit |
|----------|-------|
| Max indexes | 10 |
| Max vectors | 1M per index |
| Vector dimensions | 1536 |

## Benchmarks

- `bench_insert`: Insertion speed
- `bench_search`: Search speed
- `bench_recall`: Search recall

## Testing Requirements

- Insert/search tests
- Recall tests
- Concurrency tests

## Logging Requirements

- Operations at TRACE level
- Errors at ERROR level

## Telemetry

- Insert frequency
- Search frequency
- Average latency
- Index sizes

## Cross References

- [050_MEMORY_RUNTIME_SPEC.md](050_MEMORY_RUNTIME_SPEC.md)
- [102_EMBEDDING_SPEC.md](102_EMBEDDING_SPEC.md)
- [100_DATABASE_SCHEMA.md](100_DATABASE_SCHEMA.md)

## References

- sqlite-vec Documentation
- Vector Search Algorithms
- HNSW Index
