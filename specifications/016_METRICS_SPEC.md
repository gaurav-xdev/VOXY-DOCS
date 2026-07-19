# 016 - Metrics Specification

## Purpose

Define the metrics collection, aggregation, and export subsystem that provides observability into VOXY runtime performance and behavior.

## Scope

- Counter, gauge, and histogram metric types
- Metric collection from all subsystems
- Aggregation and windowing
- Export to Prometheus, OTLP, and Windows Performance Counters
- Metric cardinality management

## Responsibilities

- Collect metrics from all subsystems
- Aggregate metrics over time windows
- Export to configured backends
- Prevent cardinality explosion
- Support metric querying for diagnostics

## Non-Goals

- Distributed tracing (handled by Logging)
- Alerting (external system responsibility)
- Log-based metrics (use Logging subsystem)

## Architecture Position

Metrics is a cross-cutting observability service used by all subsystems.

## Inputs

- Metric recordings from subsystems
- Collection configuration
- Export backend configuration

## Outputs

- Prometheus exposition format
- OTLP protobuf messages
- Windows Performance Counter updates
- In-memory metric queries

## Interfaces

```rust
pub struct Metrics {
    pub fn counter(name: &str, labels: Labels) -> Counter;
    pub fn gauge(name: &str, labels: Labels) -> Gauge;
    pub fn histogram(name: &str, labels: Labels, buckets: Vec<f64>) -> Histogram;
    pub fn record(metric: Metric);
    pub fn export(format: ExportFormat) -> Result<String, MetricsError>;
}
```

## Data Models

### Metric

| Field | Type | Description |
|-------|------|-------------|
| name | String | Metric name (snake_case) |
| type | MetricType | Counter, Gauge, Histogram |
| labels | Map | Key-value tag pairs |
| value | f64 | Metric value |
| timestamp | DateTime | Recording time |

## State Management

- Metrics are aggregated in-memory
- No persistent storage (ephemeral)
- Export state is maintained per backend

## Threading Model

Metric recording is lock-free (atomic operations). Aggregation runs on a background task.

## IPC Requirements

- No IPC for metrics
- External processes query via HTTP /metrics endpoint

## Event Definitions

| Event | Payload | Description |
|-------|---------|-------------|
| `metrics.export` | format | Metrics exported |
| `metrics.cardinality_warning` | count | Cardinality threshold exceeded |

## Error Handling

| Code | Description | Recovery |
|------|-------------|----------|
| `MT001` | Cardinality exceeded | Drop new labels, alert |
| `MT002` | Export failure | Retry, queue for next interval |
| `MT003` | Invalid metric name | Reject, log error |

## Retry Strategy

- Export: 3 retries with exponential backoff

## Recovery Strategy

- Export backlog: Drop oldest, keep latest
- Memory pressure: Reduce aggregation windows

## Security Requirements

- Metrics do not contain PII
- /metrics endpoint requires no auth (Prometheus convention)
- Internal metrics may be sensitive; restrict access

## Configuration

```json
{
  "metrics": {
    "enabled": true,
    "collection_interval_seconds": 15,
    "export_interval_seconds": 60,
    "max_cardinality": 10000,
    "backends": ["prometheus", "otlp"],
    "prometheus_port": 9090,
    "otlp_endpoint": "http://localhost:4317"
  }
}
```

## Performance Targets

| Metric | Target |
|--------|--------|
| Record latency | < 1us |
| Export latency | < 100ms |
| Memory per metric | < 100 bytes |

## Resource Limits

| Resource | Limit |
|----------|-------|
| Active metrics | 10,000 |
| Labels per metric | 10 |
| Label value length | 128 chars |

## Benchmarks

- `bench_record`: 1M metric recordings
- `bench_export`: Full export cycle
- `bench_cardinality`: Cardinality stress test

## Testing Requirements

- Metric type accuracy tests
- Aggregation correctness tests
- Export format tests
- Cardinality limit tests

## Logging Requirements

- Export failures at WARN level
- Cardinality warnings at WARN level
- Normal operation at TRACE level

## Telemetry

- Metrics recorded per second
- Export duration
- Active metric count
- Cardinality distribution

## Cross References

- [015_LOGGING_SPEC.md](015_LOGGING_SPEC.md)
- [112_TELEMETRY_SPEC.md](112_TELEMETRY_SPEC.md)
- [120_PERFORMANCE_TARGETS.md](120_PERFORMANCE_TARGETS.md)

## References

- Prometheus Data Model
- OpenTelemetry Metrics
- Windows Performance Counters
