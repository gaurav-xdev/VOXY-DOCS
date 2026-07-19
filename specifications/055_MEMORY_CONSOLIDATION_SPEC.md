# 055 - Memory Consolidation Specification

## Purpose

Define the Memory Consolidation subsystem that periodically processes working and episodic memories, extracting patterns, merging duplicates, and promoting important information to long-term storage.

## Scope

- Working memory archiving
- Episodic memory summarization
- Semantic memory extraction
- Duplicate detection and merging
- Importance scoring updates
- Memory decay application

## Responsibilities

- Archive old working memory
- Summarize episodic events
- Extract semantic knowledge from episodes
- Detect and merge duplicates
- Update importance scores
- Apply memory decay

## Non-Goals

- Real-time memory processing
- User-facing memory management
- External memory import

## Architecture Position

Consolidation runs as a background process, transforming short-term memories into long-term knowledge.

## Inputs

- Working memory snapshots
- Episodic memory events
- Consolidation triggers (schedule or manual)

## Outputs

- Archived memories
- Summarized events
- Extracted knowledge
- Consolidation reports

## Interfaces

```rust
pub struct MemoryConsolidation {
    pub fn run_consolidation() -> Result<ConsolidationReport, ConsolidationError>;
    pub fn summarize_events(events: &[Event]) -> Result<Summary, ConsolidationError>;
    pub fn extract_knowledge(events: &[Event]) -> Vec<KnowledgeEntry>;
    pub fn merge_duplicates() -> Result<MergeReport, ConsolidationError>;
    pub fn apply_decay() -> Result<DecayReport, ConsolidationError>;
}
```

## Data Models

### ConsolidationReport

| Field | Type | Description |
|-------|------|-------------|
| archived_count | usize | Memories archived |
| summarized_count | usize | Events summarized |
| extracted_count | usize | Knowledge extracted |
| merged_count | usize | Duplicates merged |
| decayed_count | usize | Memories decayed |
| duration_ms | u64 | Processing time |

## State Management

- Consolidation state tracks last run
- Reports are stored for history
- No persistent consolidation queue

## Threading Model

Consolidation runs on blocking pool as a background task. Does not block user operations.

## IPC Requirements

- No IPC for consolidation

## Event Definitions

| Event | Payload | Description |
|-------|---------|-------------|
| `consolidation.started` | {} | Consolidation started |
| `consolidation.completed` | ConsolidationReport | Consolidation finished |
| `consolidation.error` | ConsolidationError | Error occurred |

## Error Handling

| Code | Description | Recovery |
|------|-------------|----------|
| `MC001` | Processing timeout | Resume next cycle |
| `MC002` | Storage error | Retry, alert |

## Retry Strategy

- Full consolidation: Resume on next cycle
- Individual operations: 2 retries

## Recovery Strategy

- Timeout: Resume next cycle
- Storage error: Skip batch, continue

## Security Requirements

- Consolidation runs with same privileges as memory access
- No external data exposure

## Configuration

```json
{
  "memory_consolidation": {
    "enabled": true,
    "interval_hours": 24,
    "max_processing_time_minutes": 30,
    "archive_threshold_days": 7,
    "decay_enabled": true,
    "duplicate_similarity_threshold": 0.95,
    "summary_model": "local"
  }
}
```

## Performance Targets

| Metric | Target |
|--------|--------|
| Consolidation time | < 30 minutes |
| Events per second | > 100 |
| Memory overhead | < 200MB |

## Resource Limits

| Resource | Limit |
|----------|-------|
| Processing time | 30 minutes |
| Batch size | 10,000 |
| Memory overhead | 200MB |

## Benchmarks

- `bench_consolidation`: Full cycle time
- `bench_summarization`: Summary speed
- `bench_extraction`: Knowledge extraction

## Testing Requirements

- End-to-end consolidation tests
- Summarization quality tests
- Duplicate detection tests
- Decay tests

## Logging Requirements

- Start/end at INFO level
- Progress at DEBUG level
- Errors at ERROR level

## Telemetry

- Consolidation duration
- Items processed
- Success rate

## Cross References

- [050_MEMORY_RUNTIME_SPEC.md](050_MEMORY_RUNTIME_SPEC.md)
- [052_EPISODIC_MEMORY_SPEC.md](052_EPISODIC_MEMORY_SPEC.md)
- [053_SEMANTIC_MEMORY_SPEC.md](053_SEMANTIC_MEMORY_SPEC.md)

## References

- Memory Consolidation (Neuroscience)
- ETL Patterns
- Batch Processing
