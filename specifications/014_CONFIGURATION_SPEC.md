# 014 - Configuration Specification

## Purpose

Define the configuration management subsystem responsible for loading, validating, distributing, and hot-reloading configuration across all VOXY subsystems.

## Scope

- Configuration source management (files, registry, environment)
- Schema validation and type checking
- Hot-reload without restart
- Configuration layering and overrides
- Secret redaction in logs

## Responsibilities

- Load configuration from multiple sources
- Validate against JSON Schema
- Distribute configuration to subsystems
- Support hot-reload with rollback
- Redact sensitive values in telemetry

## Non-Goals

- Configuration UI (user-facing, separate component)
- Remote configuration management (future enhancement)
- Configuration versioning (Git handles this)

## Architecture Position

Configuration is a foundational service used by all subsystems during initialization and runtime.

## Inputs

- Configuration files (JSON, TOML)
- Windows Registry (HKLM/HKCU)
- Environment variables
- Command-line arguments
- Runtime overrides

## Outputs

- Validated configuration objects
- Change notifications
- Validation errors

## Interfaces

```rust
pub struct Configuration {
    pub fn load(sources: Vec<ConfigSource>) -> Result<Config, ConfigError>;
    pub fn get<T: Deserialize>(path: &str) -> Result<T, ConfigError>;
    pub fn watch(path: &str, handler: ConfigChangeHandler) -> Watcher;
    pub fn reload() -> Result<ConfigDelta, ConfigError>;
    pub fn validate(schema: &JsonSchema) -> Result<(), ValidationError>;
}
```

## Data Models

### ConfigSource

| Source | Priority | Description |
|--------|----------|-------------|
| Default | 0 | Built-in defaults |
| File | 1 | `voxy.config.json` |
| Registry | 2 | Windows Registry |
| Environment | 3 | `VOXY_*` variables |
| CLI | 4 | Command-line args |
| Runtime | 5 | In-memory overrides |

## State Management

- Configuration is immutable after load
- Hot-reload creates new config, old is retained for rollback
- Subsystems hold references to config slices

## Threading Model

Config loads on main thread. Hot-reload runs on blocking pool. Watchers are async.

## IPC Requirements

- No IPC for configuration (in-process)
- External processes read config via IPC Protocol on request

## Event Definitions

| Event | Payload | Description |
|-------|---------|-------------|
| `config.loaded` | source | Configuration loaded |
| `config.changed` | delta | Hot-reload applied |
| `config.validation_failed` | errors | Schema validation error |
| `config.rollback` | reason | Rollback to previous config |

## Error Handling

| Code | Description | Recovery |
|------|-------------|----------|
| `CF001` | File not found | Use defaults, log warning |
| `CF002` | Parse error | Fatal, exit process |
| `CF003` | Schema violation | Fatal, exit process |
| `CF004` | Hot-reload failed | Rollback, log error |
| `CF005` | Type mismatch | Fatal, exit process |

## Retry Strategy

- File read: 3 retries with 500ms delay
- Registry read: 1 retry

## Recovery Strategy

- Hot-reload failure: Automatic rollback to previous config
- Source failure: Fall back to next priority source

## Security Requirements

- Secrets MUST be stored in Secret Storage (092), not config files
- Config files MUST have restrictive ACLs (user read-only)
- Sensitive values are redacted in logs and telemetry

## Configuration

```json
{
  "configuration": {
    "config_file_path": "%LOCALAPPDATA%/VOXY/voxy.config.json",
    "schema_validation": true,
    "hot_reload_enabled": true,
    "watch_interval_seconds": 5,
    "redact_keys": ["api_key", "secret", "password", "token"]
  }
}
```

## Performance Targets

| Metric | Target |
|--------|--------|
| Load time | < 100ms |
| Hot-reload | < 500ms |
| Get latency | < 10us |

## Resource Limits

| Resource | Limit |
|----------|-------|
| Config size | 10MB |
| Watchers | 1000 |
| Sources | 10 |

## Benchmarks

- `bench_load`: Load complex config
- `bench_hot_reload`: Reload with 100 watchers
- `bench_get`: 1M config lookups

## Testing Requirements

- Schema validation tests
- Priority override tests
- Hot-reload integration tests
- Secret redaction tests

## Logging Requirements

- Load events at INFO level
- Hot-reload at INFO level
- Validation errors at ERROR level
- Rollback at WARN level

## Telemetry

- Config load duration
- Hot-reload frequency
- Validation failure count
- Source distribution

## Cross References

- [010_KERNEL_SPEC.md](010_KERNEL_SPEC.md)
- [092_SECRET_STORAGE_SPEC.md](092_SECRET_STORAGE_SPEC.md)
- [110_CONFIGURATION_SCHEMA.md](110_CONFIGURATION_SCHEMA.md)

## References

- Rust `config` crate
- JSON Schema Draft 2020-12
- Windows Registry API
