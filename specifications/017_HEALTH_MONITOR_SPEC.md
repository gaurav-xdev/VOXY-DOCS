# 017 - Health Monitor Specification

## Purpose

Define the health checking and failure detection subsystem that monitors all VOXY subsystems, detects failures, and coordinates recovery actions.

## Scope

- Subsystem health probing
- Failure detection and classification
- Recovery coordination
- Health status aggregation
- Circuit breaker management

## Responsibilities

- Periodically probe all subsystems for health
- Detect and classify failures (transient, permanent, cascading)
- Trigger recovery via Runtime Manager
- Maintain health history
- Provide health status queries

## Non-Goals

- Root cause analysis (use Logging and Metrics)
- Automatic remediation beyond restart
- Performance profiling (Metrics responsibility)

## Architecture Position

Health Monitor consumes status from all subsystems and coordinates with Runtime Manager for recovery.

## Inputs

- Heartbeat events from subsystems
- Health check responses
- Failure events from Event Bus
- Recovery commands from Runtime Manager

## Outputs

- Health status updates to Service Registry
- Recovery requests to Runtime Manager
- Alert events to Event Bus
- Health reports for diagnostics

## Interfaces

```rust
pub struct HealthMonitor {
    pub fn register_subsystem(name: &str, check: HealthCheck) -> Result<(), HealthError>;
    pub fn report_heartbeat(name: &str, status: HealthStatus);
    pub fn get_health(name: &str) -> HealthStatus;
    pub fn get_all_health() -> Vec<(String, HealthStatus)>;
    pub fn trigger_recovery(name: &str) -> Result<(), HealthError>;
}
```

## Data Models

### HealthStatus

| Variant | Description |
|---------|-------------|
| `Healthy` | Subsystem operational |
| `Degraded` | Reduced functionality |
| `Unhealthy` | Failed, not responding |
| `Unknown` | No recent heartbeat |
| `Recovering` | Recovery in progress |

### HealthCheck

| Field | Type | Description |
|-------|------|-------------|
| interval_seconds | u64 | Probe frequency |
| timeout_seconds | u64 | Response timeout |
| retries | u32 | Retry count before failure |
| check_fn | Fn | Custom health check function |

## State Management

- Health status is maintained per subsystem
- History is retained for 24 hours
- Circuit breaker state is maintained

## Threading Model

Health checks run on dedicated Tokio tasks. One task per subsystem.

## IPC Requirements

- Health status exposed via IPC for external monitoring

## Event Definitions

| Event | Payload | Description |
|-------|---------|-------------|
| `health.status_changed` | (name, old, new) | Health status transition |
| `health.recovery_triggered` | name | Recovery initiated |
| `health.recovery_completed` | (name, success) | Recovery finished |
| `health.circuit_open` | name | Circuit breaker opened |
| `health.circuit_closed` | name | Circuit breaker closed |

## Error Handling

| Code | Description | Recovery |
|------|-------------|----------|
| `HM001` | Check timeout | Mark unhealthy, retry |
| `HM002` | Recovery failed | Escalate, alert |
| `HM003` | Unknown subsystem | Log warning |

## Retry Strategy

- Health check: 3 retries, 1s intervals
- Recovery: 3 attempts, 5s intervals

## Recovery Strategy

- Transient failure: Wait, retry check
- Permanent failure: Trigger subsystem restart
- Cascading failure: Enter degraded mode

## Security Requirements

- Health checks do not expose sensitive data
- Recovery actions require capability tokens

## Configuration

```json
{
  "health_monitor": {
    "check_interval_seconds": 10,
    "check_timeout_seconds": 5,
    "max_retries": 3,
    "recovery_enabled": true,
    "circuit_breaker": {
      "failure_threshold": 5,
      "recovery_timeout_seconds": 30
    },
    "history_retention_hours": 24
  }
}
```

## Performance Targets

| Metric | Target |
|--------|--------|
| Check latency | < 100ms |
| Detection time | < 15s |
| Recovery initiation | < 5s |

## Resource Limits

| Resource | Limit |
|----------|-------|
| Concurrent checks | 50 |
| History entries | 10,000 |

## Benchmarks

- `bench_check_latency`: Single check latency
- `bench_detection`: Failure detection time
- `bench_recovery`: Recovery cycle duration

## Testing Requirements

- Health state transition tests
- Recovery success/failure tests
- Circuit breaker tests
- Cascading failure tests

## Logging Requirements

- Status changes at INFO level
- Recovery at INFO level
- Failures at WARN level
- Circuit events at WARN level

## Telemetry

- Health check duration
- Status change frequency
- Recovery success rate
- Circuit breaker events

## Cross References

- [011_RUNTIME_MANAGER_SPEC.md](011_RUNTIME_MANAGER_SPEC.md)
- [013_SERVICE_REGISTRY_SPEC.md](013_SERVICE_REGISTRY_SPEC.md)
- [016_METRICS_SPEC.md](016_METRICS_SPEC.md)

## References

- Health Check Pattern (Microservices)
- Circuit Breaker Pattern (Release It!)
- Kubernetes Health Probes
