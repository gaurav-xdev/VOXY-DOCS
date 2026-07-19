# 052 - Episodic Memory Specification

## Purpose

Define the Episodic Memory subsystem that stores event-based memories of user interactions, system actions, and significant occurrences with temporal context.

## Scope

- Event recording and storage
- Temporal indexing
- Event retrieval by time, context, or similarity
- Event summarization
- Memory decay and importance scoring

## Responsibilities

- Record significant events
- Index events temporally
- Retrieve events by time range
- Retrieve events by semantic similarity
- Score event importance
- Apply memory decay

## Non-Goals

- General knowledge storage (Semantic Memory)
- Skill storage (Procedural Memory)
- Real-time event streaming

## Architecture Position

Episodic Memory stores the "what happened" of VOXY's experience. It is queried by Context Assembly for relevant past events.

## Inputs

- Events from Event Bus
- Importance scores
- User feedback

## Outputs

- Retrieved events
- Temporal summaries
- Event statistics

## Interfaces

```rust
pub struct EpisodicMemory {
    pub fn record_event(event: Event) -> Result<EventId, MemoryError>;
    pub fn query_by_time(range: TimeRange) -> Vec<Event>;
    pub fn query_by_similarity(query: &str, top_k: usize) -> Vec<Event>;
    pub fn get_summary(range: TimeRange) -> String;
    pub fn set_importance(id: EventId, score: f32);
}
```

## Data Models

### Event

| Field | Type | Description |
|-------|------|-------------|
| id | EventId | Unique identifier |
| type | String | Event type |
| description | String | Human-readable description |
| timestamp | DateTime | Event time |
| embedding | Vec<f32> | Semantic embedding |
| importance | f32 | Importance score (0-1) |
| metadata | Map | Additional data |
| related_events | Vec<EventId> | Linked events |

## State Management

- Events stored in SQLite with sqlite-vec for vector search
- Importance scores updated dynamically
- Decay applied periodically

## Threading Model

Event recording is async. Queries run on blocking pool.

## IPC Requirements

- No IPC for episodic memory

## Event Definitions

| Event | Payload | Description |
|-------|---------|-------------|
| `episodic.event_recorded` | Event | Event stored |
| `episodic.decay_applied` | count | Decay cycle complete |
| `episodic.importance_updated` | (id, score) | Importance changed |

## Error Handling

| Code | Description | Recovery |
|------|-------------|----------|
| `EM001` | Storage full | Archive old events |
| `EM002` | Embedding failed | Store without embedding |

## Retry Strategy

- Store: 2 retries
- Query: 1 retry

## Recovery Strategy

- Storage full: Archive by importance
- Embedding failure: Store text only

## Security Requirements

- Events encrypted at rest
- Access controlled by capability tokens

## Configuration

```json
{
  "episodic_memory": {
    "max_events": 100000,
    "decay_interval_hours": 24,
    "decay_rate": 0.01,
    "importance_threshold": 0.3,
    "archive_after_days": 365
  }
}
```

## Performance Targets

| Metric | Target |
|--------|--------|
| Record latency | < 20ms |
| Query latency | < 100ms |
| Decay cycle | < 1 minute |

## Resource Limits

| Resource | Limit |
|----------|-------|
| Max events | 100,000 |
| Event size | 10KB |
| Query results | 100 |

## Benchmarks

- `bench_record`: Event storage
- `bench_query`: Retrieval speed
- `bench_decay`: Decay performance

## Testing Requirements

- Event recording tests
- Temporal query tests
- Similarity search tests
- Decay tests

## Logging Requirements

- Recording at DEBUG level
- Queries at TRACE level

## Telemetry

- Events per hour
- Query latency
- Storage size
- Decay rate

## Cross References

- [050_MEMORY_RUNTIME_SPEC.md](050_MEMORY_RUNTIME_SPEC.md)
- [055_MEMORY_CONSOLIDATION_SPEC.md](055_MEMORY_CONSOLIDATION_SPEC.md)
- [056_MEMORY_RETRIEVAL_SPEC.md](056_MEMORY_RETRIEVAL_SPEC.md)

## References

- Episodic Memory (Cognitive Psychology)
- Temporal Database Design
- Vector Search (sqlite-vec)
