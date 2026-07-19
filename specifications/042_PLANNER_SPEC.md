# 041 - Model Router Specification

## Purpose

Define the Model Router subsystem that selects the optimal AI model for each inference request, balancing capability, latency, cost, and privacy requirements across local and cloud providers.

## Scope

- Model selection and routing
- Provider health monitoring
- Load balancing across providers
- Fallback chain management
- Token usage tracking
- Cost optimization
- Local vs cloud decision logic

## Responsibilities

- Route requests to appropriate models
- Monitor provider health and latency
- Implement fallback chains
- Track token usage and costs
- Optimize for latency vs quality
- Support local-first inference

## Non-Goals

- Model training or fine-tuning
- Prompt engineering (Context Assembly)
- Result post-processing (Executor)

## Architecture Position

Model Router sits between Context Assembly and external AI providers. It is the gateway for all LLM inference.

## Inputs

- Assembled context windows
- Model preference hints
- Capability requirements
- Privacy constraints
- Cost budgets

## Outputs

- LLM responses
- Token usage reports
- Routing decisions
- Provider metrics

## Interfaces

```rust
pub struct ModelRouter {
    pub fn route(request: InferenceRequest) -> Result<InferenceResponse, RouterError>;
    pub fn get_available_models() -> Vec<ModelInfo>;
    pub fn get_provider_health() -> Vec<ProviderHealth>;
    pub fn set_preference(preference: ModelPreference);
}
```

## Data Models

### ModelInfo

| Field | Type | Description |
|-------|------|-------------|
| id | String | Model identifier |
| name | String | Human-readable name |
| provider | Provider | Cloud or local |
| capabilities | Vec<String> | Supported capabilities |
| max_tokens | usize | Context window size |
| latency_p50_ms | u64 | Median latency |
| cost_per_1k_tokens | f32 | Cost in USD |
| privacy | PrivacyLevel | Local, Cloud, Hybrid |

### InferenceRequest

| Field | Type | Description |
|-------|------|-------------|
| context | ContextWindow | Assembled context |
| max_tokens | usize | Max response tokens |
| temperature | f32 | Sampling temperature |
| tools | Option<Vec<ToolSchema>> | Available tools |
| priority | Priority | Request priority |

### InferenceResponse

| Field | Type | Description |
|-------|------|-------------|
| content | String | Model output |
| model_used | String | Actual model used |
| tokens_used | TokenUsage | Input/output tokens |
| latency_ms | u64 | Response latency |
| finish_reason | String | Stop, length, tool_calls |

### Provider

| Variant | Description |
|---------|-------------|
| `OpenAI` | OpenAI GPT models |
| `Anthropic` | Claude models |
| `Azure` | Azure OpenAI |
| `Local` | Local ONNX/Phi Silica |
| `Ollama` | Ollama local server |

## State Management

- Provider health is maintained in-memory
- Token usage is tracked per provider
- Routing decisions are logged

## Threading Model

Async requests to cloud providers. Local inference runs on blocking pool.

## IPC Requirements

- No IPC for inference
- Provider status may be queried via IPC

## Event Definitions

| Event | Payload | Description |
|-------|---------|-------------|
| `router.request` | InferenceRequest | Request routed |
| `router.response` | InferenceResponse | Response received |
| `router.fallback` | (primary, fallback) | Fallback triggered |
| `router.provider.unhealthy` | provider | Provider marked unhealthy |

## Error Handling

| Code | Description | Recovery |
|------|-------------|----------|
| `MR001` | Provider timeout | Fallback to next provider |
| `MR002` | Rate limit | Backoff, fallback |
| `MR003` | Invalid model | Use default model |
| `MR004` | Context too large | Truncate, retry |
| `MR005` | All providers failed | Return error |

## Retry Strategy

- Provider call: 2 retries per provider
- Fallback: Immediate to next provider
- Rate limit: Exponential backoff

## Recovery Strategy

- Single provider failure: Use fallback
- All providers failure: Queue for retry, alert user
- Context too large: Compress and retry

## Security Requirements

- API keys in Secret Storage
- Local inference preferred for sensitive data
- Cloud requests use HTTPS
- No data logging by providers without consent

## Configuration

```json
{
  "model_router": {
    "primary_provider": "Local",
    "fallback_chain": ["Azure", "OpenAI", "Anthropic"],
    "default_model": "phi-silica",
    "max_latency_ms": 5000,
    "enable_cost_tracking": true,
    "privacy_mode": "local_first",
    "token_budget_daily": 100000,
    "health_check_interval_seconds": 60
  }
}
```

## Performance Targets

| Metric | Target |
|--------|--------|
| Routing latency | < 10ms |
| First token latency | < 500ms |
| Provider failover | < 100ms |
| Token throughput | > 100 tokens/sec |

## Resource Limits

| Resource | Limit |
|----------|-------|
| Concurrent requests | 10 |
| Queue depth | 100 |
| Token budget | Configurable |

## Benchmarks

- `bench_routing_latency`: Decision time
- `bench_inference_latency`: End-to-end
- `bench_failover`: Failover time

## Testing Requirements

- Provider fallback tests
- Load balancing tests
- Cost tracking tests
- Privacy mode tests

## Logging Requirements

- Routing decisions at DEBUG level
- Provider failures at WARN level
- Token usage at INFO level

## Telemetry

- Requests per provider
- Latency per provider
- Token usage
- Cost per request
- Fallback frequency

## Cross References

- [040_CONTEXT_ASSEMBLY_SPEC.md](040_CONTEXT_ASSEMBLY_SPEC.md)
- [042_PLANNER_SPEC.md](042_PLANNER_SPEC.md)
- [045_TOOL_CALLING_SPEC.md](045_TOOL_CALLING_SPEC.md)
- [092_SECRET_STORAGE_SPEC.md](092_SECRET_STORAGE_SPEC.md)

## References

- OpenAI API
- Anthropic API
- Azure OpenAI Service
- Windows AI APIs (Phi Silica)
- ONNX Runtime
