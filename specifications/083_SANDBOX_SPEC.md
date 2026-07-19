# 083 - Sandbox Specification

## Purpose

Define the sandbox subsystem that isolates third-party plugins in restricted processes with limited access to system resources.

## Scope

- Process isolation
- Resource limits
- IPC-only communication
- File system virtualization
- Network restrictions
- Memory limits

## Responsibilities

- Spawn sandboxed processes
- Enforce resource limits
- Mediate all plugin I/O
- Monitor sandbox health
- Handle sandbox crashes
- Provide escape hatch for debugging

## Non-Goals

- Containerization (process-level only)
- Cross-platform sandbox
- Full virtualization

## Architecture Position

Sandbox wraps Plugin Runtime processes. All plugin code runs inside the sandbox.

## Sandbox Architecture

```
[VOXY Core Process] <--IPC--> [Sandbox Broker] <--IPC--> [Plugin Process (Low Integrity)]
                                    |
                              [Resource Monitor]
```

## Data Models

### SandboxConfig

| Field | Type | Description |
|-------|------|-------------|
| max_memory_mb | usize | Memory limit |
| max_cpu_percent | f32 | CPU limit |
| max_file_size_mb | usize | File write limit |
| allowed_directories | Vec<Path> | Writable directories |
| network_allowed | bool | Network access |
| process_spawn_allowed | bool | Process spawning |

### SandboxProcess

| Field | Type | Description |
|-------|------|-------------|
| pid | u32 | Process ID |
| plugin_id | String | Plugin identifier |
| start_time | DateTime | Start time |
| memory_usage_mb | f64 | Current memory |
| cpu_percent | f32 | Current CPU |

## Interfaces

```rust
pub struct Sandbox {
    pub fn spawn(plugin: &Plugin, config: SandboxConfig) -> Result<SandboxProcess, SandboxError>;
    pub fn terminate(process: &SandboxProcess);
    pub fn get_stats(process: &SandboxProcess) -> SandboxStats;
    pub fn enforce_limits(process: &SandboxProcess);
}
```

## State Management

- Sandbox processes tracked in-memory
- Resource usage monitored
- No persistent state

## Threading Model

Sandbox broker runs as a separate thread. Resource monitoring runs periodically.

## IPC Requirements

- All plugin communication via IPC Protocol
- Broker mediates all requests

## Event Definitions

| Event | Payload | Description |
|-------|---------|-------------|
| `sandbox.spawned` | SandboxProcess | Process spawned |
| `sandbox.terminated` | pid | Process terminated |
| `sandbox.limit_exceeded` | (pid, limit) | Limit exceeded |
| `sandbox.crashed` | (pid, error) | Process crashed |

## Error Handling

| Code | Description | Recovery |
|------|-------------|----------|
| `SB001` | Spawn failed | Retry once |
| `SB002` | Memory limit exceeded | Terminate |
| `SB003` | CPU limit exceeded | Throttle |
| `SB004` | IPC failure | Reconnect |

## Retry Strategy

- Spawn: 1 retry
- IPC: 3 retries

## Recovery Strategy

- Limit exceeded: Terminate or throttle
- Crash: Restart once, then disable
- IPC failure: Reconnect

## Security Requirements

- Plugins run at Low Integrity Level
- No access to VOXY core memory
- All file access mediated
- Network filtered
- No registry write

## Configuration

```json
{
  "sandbox": {
    "max_memory_mb": 256,
    "max_cpu_percent": 25.0,
    "max_file_size_mb": 100,
    "allowed_directories": ["%LOCALAPPDATA%/VOXY/plugins/data/"],
    "network_allowed": false,
    "process_spawn_allowed": false,
    "monitor_interval_seconds": 5,
    "auto_terminate_on_limit": true
  }
}
```

## Performance Targets

| Metric | Target |
|--------|--------|
| Spawn time | < 1s |
| IPC latency | < 5ms |
| Monitor overhead | < 1% CPU |

## Resource Limits

| Resource | Limit |
|----------|-------|
| Sandboxed processes | 50 |
| Memory per process | 256MB |
| CPU per process | 25% |

## Benchmarks

- `bench_spawn`: Spawn time
- `bench_ipc`: IPC latency
- `bench_overhead`: Sandbox overhead

## Testing Requirements

- Spawn/terminate tests
- Limit enforcement tests
- IPC tests
- Crash recovery tests

## Logging Requirements

- Spawns at INFO level
- Limits at WARN level
- Crashes at ERROR level

## Telemetry

- Active sandboxes
- Resource usage
- Crash frequency
- IPC latency

## Cross References

- [080_PLUGIN_RUNTIME_SPEC.md](080_PLUGIN_RUNTIME_SPEC.md)
- [082_PERMISSION_MODEL_SPEC.md](082_PERMISSION_MODEL_SPEC.md)
- [090_SECURITY_MODEL_SPEC.md](090_SECURITY_MODEL_SPEC.md)

## References

- Windows Integrity Levels
- Chrome Sandbox
- Windows Job Objects
