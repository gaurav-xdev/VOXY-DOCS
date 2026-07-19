# 102 - Embedding Specification

## Purpose

Define the Embedding subsystem that generates vector representations of text for semantic search and similarity comparison.

## Scope

- Text embedding generation
- Model management
- Batch processing
- Embedding caching
- Dimension normalization
- Provider abstraction

## Responsibilities

- Generate text embeddings
- Manage embedding models
- Support batch processing
- Cache embeddings
- Normalize vectors
- Abstract provider differences

## Non-Goals

- Image embeddings (Vision Runtime)
- Audio embeddings (Audio Pipeline)
- Model training

## Architecture Position

Embedding sits between Memory subsystems and Vector Database, generating vectors for storage and search.

## Data Models

### EmbeddingModel

| Field | Type | Description |
|-------|------|-------------|
| id | String | Model identifier |
| name | String | Model name |
| dimensions | usize | Output dimensions |
| provider | Provider | Local or cloud |
| max_tokens | usize | Max input tokens |

### EmbeddingRequest

| Field | Type | Description |
|-------|------|-------------|
| texts | Vec<String> | Texts to embed |
| model | String | Model to use |
| normalize | bool | Normalize output |

### EmbeddingResult

| Field | Type | Description |
|-------|------|-------------|
| embeddings | Vec<Vec<f32>> | Generated vectors |
| model_used | String | Actual model |
| tokens_used | usize | Tokens consumed |

## Interfaces

```rust
pub struct Embedding {
    pub fn embed(request: EmbeddingRequest) -> Result<EmbeddingResult, EmbeddingError>;
    pub fn get_model(model_id: &str) -> Option<EmbeddingModel>;
    pub fn list_models() -> Vec<EmbeddingModel>;
    pub fn cache_size() -> usize;
}
```

## Providers

| Provider | Model | Dimensions | Local |
|----------|-------|-----------|-------|
| Local | all-MiniLM-L6-v2 | 384 | Yes |
| Local | all-mpnet-base-v2 | 768 | Yes |
| OpenAI | text-embedding-3-small | 1536 | No |
| OpenAI | text-embedding-3-large | 3072 | No |

## State Management

- Models loaded on demand
- Cache in-memory with TTL
- No persistent state

## Threading Model

Local inference on blocking pool. Cloud requests async.

## IPC Requirements

- No IPC

## Event Definitions

| Event | Payload | Description |
|-------|---------|-------------|
| `embedding.generated` | EmbeddingResult | Embedding done |
| `embedding.cache_hit` | text | Cache hit |
| `embedding.model_loaded` | model | Model loaded |

## Error Handling

| Code | Description | Recovery |
|------|-------------|----------|
| `EM001` | Model load failure | Use fallback |
| `EM002` | Text too long | Truncate |
| `EM003` | Provider timeout | Retry |

## Configuration

```json
{
  "embedding": {
    "default_model": "all-MiniLM-L6-v2",
    "fallback_model": "all-mpnet-base-v2",
    "cache_enabled": true,
    "cache_size": 10000,
    "cache_ttl_seconds": 3600,
    "batch_size": 32,
    "normalize": true
  }
}
```

## Performance Targets

| Metric | Target |
|--------|--------|
| Single embedding | < 50ms |
| Batch embedding | < 500ms (32 items) |
| Cache hit rate | > 70% |

## Resource Limits

| Resource | Limit |
|----------|-------|
| Model memory | 500MB |
| Cache entries | 10,000 |
| Max text length | 512 tokens |

## Benchmarks

- `bench_single`: Single embedding speed
- `bench_batch`: Batch embedding speed
- `bench_cache`: Cache hit rate

## Testing Requirements

- Accuracy tests
- Batch tests
- Cache tests
- Fallback tests

## Logging Requirements

- Generation at TRACE level
- Errors at ERROR level

## Telemetry

- Generation frequency
- Average latency
- Cache hit rate
- Model usage

## Cross References

- [050_MEMORY_RUNTIME_SPEC.md](050_MEMORY_RUNTIME_SPEC.md)
- [101_VECTOR_DATABASE_SPEC.md](101_VECTOR_DATABASE_SPEC.md)
- [053_SEMANTIC_MEMORY_SPEC.md](053_SEMANTIC_MEMORY_SPEC.md)

## References

- Sentence Transformers
- ONNX Runtime
- OpenAI Embeddings API
