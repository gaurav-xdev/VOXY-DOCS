# 015 - Logging Specification

## Purpose

Define the structured logging subsystem that provides consistent, queryable, and actionable log output across all VOXY subsystems.

## Scope

- Structured JSON log output
- Log level management per subsystem
- Contextual field injection (trace IDs, subsystem, correlation)
- Log rotation and retention
- Integration with Windows Event Log

## Responsibilities

- Produce structured, machine-readable logs
- Support dynamic log level changes
- Correlate logs across async boundaries
- Rotate log files to prevent disk exhaustion
- Forward critical events to Windows Event Log

## Non-Goals

- Log aggregation (external SIEM responsibility)
- Log analysis UI
- Real-time log streaming (use Event Bus)

## Architecture Position

Logging is a cross-cutting concern used by all subsystems. It integrates with the Event Bus for critical alerts.

## Inputs

- Log events from all subsystems
- Log level configuration from Configuration
- Context from tracing spans

## Outputs

- JSON log files
- Windows Event Log entries
- Console output (development mode)

## Interfaces

```rust
pub struct Logger {
    pub fn init(config: LogConfig) -> Result<(), LogError>;
    pub fn set_level(subsystem: &str, level: Level);
    pub fn log(event: LogEvent);
    pub fn flush();
}
```

## Data Models

### LogEvent

| Field | Type | Description |
|-------|------|-------------|
| timestamp | ISO8601 | Event time |
| level | Level | TRACE, DEBUG, INFO, WARN, ERROR |
| subsystem | String | Source subsystem |
| message | String | Human-readable message |
| fields | Map | Structured key-value data |
| trace_id | UUID | Distributed trace ID |
| span_id | UUID | Current span ID |
| file | String | Source file |
| line | u32 | Source line |

## State Management

- Log levels are mutable at runtime
- File handles are managed by rotation logic
- No persistent state

## Threading Model

Logging is async-safe. All log events are queued and processed by a dedicated writer thread.

## IPC Requirements

- No IPC for logging
- Critical errors (ERROR level) are published to Event Bus

## Event Definitions

| Event | Payload | Description |
|-------|---------|-------------|
| `log.level_changed` | (subsystem, old, new) | Log level changed |
| `log.rotation` | (old_file, new_file) | Log file rotated |
| `log.critical` | LogEvent | Critical error forwarded to Event Bus |

## Error Handling

| Code | Description | Recovery |
|------|-------------|----------|
| `LG001` | File write failure | Retry, then stderr fallback |
| `LG002` | Disk full | Drop logs, alert |
| `LG003` | Event Log write failure | Log to file only |

## Retry Strategy

- File write: 3 retries with 100ms delay
- Event Log: 1 retry

## Recovery Strategy

- Disk full: Switch to stderr, alert user
- File corruption: Rotate immediately

## Security Requirements

- Secrets MUST be redacted before logging
- Log files MUST have user-only ACLs
- Windows Event Log entries for audit events

## Configuration

```json
{
  "logging": {
    "level": "INFO",
    "format": "json",
    "output_path": "%LOCALAPPDATA%/VOXY/logs/",
    "rotation": {
      "max_size_mb": 100,
      "max_files": 10,
      "max_age_days": 30
    },
    "console": false,
    "windows_event_log": true,
    "redact_patterns": ["password", "token", "secret", "api_key"]
  }
}
```

## Performance Targets

| Metric | Target |
|--------|--------|
| Log latency (p99) | < 100us |
| Throughput | > 50,000 logs/sec |
| Rotation time | < 50ms |

## Resource Limits

| Resource | Limit |
|----------|-------|
| Log buffer | 10MB |
| Queue depth | 100,000 |
| File size | 100MB |

## Benchmarks

- `bench_log_latency`: Single thread logging
- `bench_throughput`: Max logs per second
- `bench_rotation`: Rotation under load

## Testing Requirements

- Log level filtering tests
- Redaction tests
- Rotation tests
- Windows Event Log integration tests

## Logging Requirements

- Self-logging at DEBUG level
- Rotation events at INFO level
- Errors at ERROR level

## Telemetry

- Logs per second by subsystem
- Log level distribution
- Rotation frequency
- Buffer overflow count

## Cross References

- [012_EVENT_BUS_SPEC.md](012_EVENT_BUS_SPEC.md)
- [014_CONFIGURATION_SPEC.md](014_CONFIGURATION_SPEC.md)
- [112_TELEMETRY_SPEC.md](112_TELEMETRY_SPEC.md)

## References

- Rust `tracing` crate
- Rust `tracing-subscriber` crate
- Windows Event Log API
