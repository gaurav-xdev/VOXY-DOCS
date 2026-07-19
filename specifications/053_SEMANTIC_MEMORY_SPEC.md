# 053 - Semantic Memory Specification

## Purpose

Define the Semantic Memory subsystem that stores general knowledge, facts, and learned information in a structured, queryable format with vector embeddings for similarity search.

## Scope

- Knowledge entry storage
- Vector embedding indexing
- Semantic similarity search
- Knowledge graph relationships
- Entity extraction and linking
- Knowledge consolidation

## Responsibilities

- Store factual knowledge
- Index knowledge with embeddings
- Support semantic search
- Maintain knowledge relationships
- Extract entities from new information
- Consolidate duplicate knowledge

## Non-Goals

- Event storage (Episodic Memory)
- Skill storage (Procedural Memory)
- Real-time knowledge acquisition

## Architecture Position

Semantic Memory stores the "what is known" of VOXY. It provides factual context for reasoning.

## Inputs

- Knowledge entries from cognitive processing
- Entity extractions
- User-provided facts
- External knowledge imports

## Outputs

- Retrieved knowledge
- Entity information
- Knowledge graph queries
- Similarity search results

## Interfaces

```rust
pub struct SemanticMemory {
    pub fn store_knowledge(entry: KnowledgeEntry) -> Result<KnowledgeId, MemoryError>;
    pub fn search(query: &str, top_k: usize) -> Vec<KnowledgeEntry>;
    pub fn search_by_embedding(embedding: &[f32], top_k: usize) -> Vec<KnowledgeEntry>;
    pub fn get_entity(name: &str) -> Option<Entity>;
    pub fn get_related(id: KnowledgeId) -> Vec<KnowledgeEntry>;
}
```

## Data Models

### KnowledgeEntry

| Field | Type | Description |
|-------|------|-------------|
| id | KnowledgeId | Unique identifier |
| content | String | Knowledge text |
| embedding | Vec<f32> | Vector embedding |
| entities | Vec<Entity> | Extracted entities |
| source | String | Knowledge source |
| confidence | f32 | Confidence score |
| created_at | DateTime | Creation time |
| updated_at | DateTime | Last update |

### Entity

| Field | Type | Description |
|-------|------|-------------|
| name | String | Entity name |
| type | String | Entity type |
| mentions | Vec<KnowledgeId> | Mentioning entries |

## State Management

- Knowledge stored in SQLite with sqlite-vec
- Entity index maintained separately
- Relationships stored as graph edges

## Threading Model

Storage and search run on blocking pool. Entity extraction may use async.

## IPC Requirements

- No IPC for semantic memory

## Event Definitions

| Event | Payload | Description |
|-------|---------|-------------|
| `semantic.knowledge_stored` | KnowledgeEntry | Knowledge stored |
| `semantic.entity_extracted` | Entity | Entity found |
| `semantic.consolidated` | count | Duplicates merged |

## Error Handling

| Code | Description | Recovery |
|------|-------------|----------|
| `SM001` | Embedding failed | Store without vector |
| `SM002` | Duplicate detected | Merge or skip |

## Retry Strategy

- Store: 2 retries
- Search: 1 retry

## Recovery Strategy

- Embedding failure: Store text only
- Duplicate: Merge content

## Security Requirements

- Knowledge encrypted at rest
- Access controlled

## Configuration

```json
{
  "semantic_memory": {
    "max_entries": 500000,
    "embedding_dimensions": 384,
    "similarity_threshold": 0.85,
    "enable_entity_extraction": true,
    "consolidation_interval_hours": 168
  }
}
```

## Performance Targets

| Metric | Target |
|--------|--------|
| Store latency | < 50ms |
| Search latency | < 100ms |
| Entity extraction | < 200ms |

## Resource Limits

| Resource | Limit |
|----------|-------|
| Max entries | 500,000 |
| Entry size | 10KB |
| Entities | 100,000 |

## Benchmarks

- `bench_store`: Storage speed
- `bench_search`: Search speed
- `bench_extraction`: Entity extraction

## Testing Requirements

- Storage tests
- Similarity search tests
- Entity extraction tests
- Consolidation tests

## Logging Requirements

- Storage at DEBUG level
- Search at TRACE level

## Telemetry

- Entries stored
- Search frequency
- Entity count
- Consolidation events

## Cross References

- [050_MEMORY_RUNTIME_SPEC.md](050_MEMORY_RUNTIME_SPEC.md)
- [055_MEMORY_CONSOLIDATION_SPEC.md](055_MEMORY_CONSOLIDATION_SPEC.md)
- [056_MEMORY_RETRIEVAL_SPEC.md](056_MEMORY_RETRIEVAL_SPEC.md)
- [102_EMBEDDING_SPEC.md](102_EMBEDDING_SPEC.md)

## References

- Semantic Memory (Cognitive Psychology)
- Knowledge Graphs
- Vector Databases
- Named Entity Recognition
