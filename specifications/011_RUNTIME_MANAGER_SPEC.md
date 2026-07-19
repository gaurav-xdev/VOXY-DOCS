# 011 - Runtime Manager Specification

## Purpose

Define the runtime orchestration layer that manages subsystem lifecycle, coordinates runtime state transitions, and provides a unified control plane for VOXY operations.

## Scope

- Subsystem lifecycle orchestration (start, stop, restart)
- Runtime state machine management
- Dependency-aware operation scheduling
- Graceful degradation handling
- Runtime reconfiguration (hot-reload)

## Responsibilities

- Coordinate subsystem start/stop operations respecting dependency graphs
- Manage runtime state transitions (Running, Paused, Degraded, Stopped)
- Handle runtime reconfiguration without full restart
- Implement circuit breaker patterns for failing subsystems
- Provide runtime control API (pause, resume, restart subsystem)

## Non-Goals

- Low-level process management (Kernel responsibility)
- Direct subsystem implementation (delegated to subsystems)
- User-facing control interface (UI layer responsibility)

## Architecture Position

The Runtime Manager sits above the Kernel and below individual subsystems. It consumes Kernel services and exposes lifecycle management to the Event Bus and Service Registry.

## Inputs

- Subsystem registration requests from Kernel
- Lifecycle commands from Event Bus (`runtime.command.*`)
- Health status updates from Health Monitor
- Configuration change events from Configuration Manager

## Outputs

- Subsystem state change notifications
- Runtime state broadcasts
- Degradation reports
- Reconfiguration status

## Interfaces

### Public Contracts

```rust
pub struct RuntimeManager {
    pub fn start_subsystem(name: &str) -> Result<SubsystemState, RuntimeError>;
    pub fn stop_subsystem(name: &str, mode: StopMode) -> Result<(), RuntimeError>;
    pub fn restart_subsystem(name: &str) -> Result<SubsystemState, RuntimeError>;
    pub fn get_runtime_state() -> RuntimeState;
    pub fn pause_runtime() -> Result<(), RuntimeError>;
    pub fn resume_runtime() -> Result<(), RuntimeError>;
    pub fn apply_config(delta: ConfigDelta) -> Result<ConfigApplyResult, RuntimeError>;
}
```

### Internal Contracts

- Subsystem start order MUST respect dependency graph (topological sort)
- Stop order MUST be reverse dependency order
- Parallel start is permitted for subsystems with no inter-dependencies
- Maximum parallel starts: 4

## Data Models

### RuntimeState

| Variant | Description |
|---------|-------------|
| `Initializing` | Bootstrapping in progress |
| `Running` | All critical subsystems operational |
| `Paused` | Intentionally suspended |
| `Degraded` | Some non-critical subsystems failed |
| `Recovering` | Attempting recovery from degraded state |
| `Stopping` | Graceful shutdown in progress |
| `Stopped` | All subsystems terminated |

### SubsystemState

| Variant | Description |
|---------|-------------|
| `Registered` | Known to Runtime Manager, not started |
| `Starting` | Initialization in progress |
| `Running` | Fully operational |
| `Paused` | Temporarily suspended |
| `Stopping` | Shutdown in progress |
| `Stopped` | Not running |
| `Failed` | Initialization or runtime failure |
| `Degraded` | Running with reduced functionality |

## State Management

### Runtime State Machine

```
[Initializing] -> [Running]
[Running] <-> [Paused]
[Running] -> [Degraded] -> [Recovering] -> [Running]
[Running] -> [Stopping] -> [Stopped]
[Degraded] -> [Stopping] -> [Stopped]
```

## Threading Model

Runtime Manager operations run on the Tokio runtime. State transitions are serialized through a single `Mutex<RuntimeState>` to prevent races. Subsystem operations are async and may run concurrently.

## IPC Requirements

- Receives commands via Event Bus (`runtime.command.*`)
- Publishes state changes via Event Bus (`runtime.state.*`)
- No direct IPC protocol involvement

## Event Definitions

| Event | Payload | Description |
|-------|---------|-------------|
| `runtime.state.changed` | (old, new, reason) | Runtime state transition |
| `runtime.subsystem.state` | (name, old, new) | Subsystem state transition |
| `runtime.command.start` | subsystem_name | Request to start subsystem |
| `runtime.command.stop` | (name, mode) | Request to stop subsystem |
| `runtime.command.restart` | subsystem_name | Request to restart subsystem |
| `runtime.degradation` | (subsystem, error) | Subsystem entered failed state |
| `runtime.recovery` | subsystem_name | Subsystem recovery initiated |

## Error Handling

### RuntimeError Taxonomy

| Code | Description | Recovery |
|------|-------------|----------|
| `RM001` | Subsystem not found | Fatal to command, log error |
| `RM002` | Dependency not satisfied | Queue start, retry when deps ready |
| `RM003` | Start timeout | Mark failed, trigger degradation |
| `RM004` | Stop timeout | Force stop, log warning |
| `RM005` | Invalid state transition | Reject command, log error |
| `RM006` | Config apply failed | Rollback, report failure |

## Retry Strategy

- Subsystem start: 2 retries, 2s and 4s delays
- Recovery from degraded: 3 retries, 5s intervals
- Config hot-reload: 1 retry, immediate rollback on failure

## Recovery Strategy

- **Subsystem Failure**: Isolate failed subsystem, notify Health Monitor, attempt restart if auto-recovery enabled
- **Cascading Failure**: If critical subsystem fails, enter Stopping state
- **Config Failure**: Rollback to previous config, notify user

## Security Requirements

- Runtime commands MUST be authenticated (capability token)
- Subsystem isolation enforced via process boundaries where applicable
- No elevation for runtime operations

## Configuration

```json
{
  "runtime_manager": {
    "auto_recovery": true,
    "recovery_max_retries": 3,
    "recovery_backoff_seconds": 5,
    "start_timeout_seconds": 30,
    "stop_timeout_seconds": 10,
    "max_parallel_starts": 4,
    "degraded_mode_enabled": true
  }
}
```

## Performance Targets

| Metric | Target |
|--------|--------|
| Subsystem start latency | < 5s |
| Subsystem stop latency | < 3s |
| State transition latency | < 100ms |
| Config hot-reload | < 2s |

## Resource Limits

| Resource | Limit |
|----------|-------|
| Concurrent subsystem ops | 8 |
| Pending command queue | 100 |
| State history buffer | 1000 entries |

## Benchmarks

- `bench_runtime_startup`: Full runtime initialization
- `bench_subsystem_restart`: Stop + start cycle
- `bench_state_transition`: Single state change latency

## Testing Requirements

- State machine property tests (all valid transitions)
- Dependency resolution tests
- Chaos engineering (random subsystem kills)
- Hot-reload integration tests

## Logging Requirements

- All state transitions at INFO level
- Subsystem operations at DEBUG level
- Degradation events at WARN level
- Recovery attempts at INFO level

## Telemetry

- State transition frequency
- Subsystem start/stop duration
- Degradation/recovery events
- Config reload success rate

## Cross References

- [010_KERNEL_SPEC.md](010_KERNEL_SPEC.md)
- [012_EVENT_BUS_SPEC.md](012_EVENT_BUS_SPEC.md)
- [013_SERVICE_REGISTRY_SPEC.md](013_SERVICE_REGISTRY_SPEC.md)
- [017_HEALTH_MONITOR_SPEC.md](017_HEALTH_MONITOR_SPEC.md)
- [014_CONFIGURATION_SPEC.md](014_CONFIGURATION_SPEC.md)

## References

- Tokio Task Management
- Windows Service Control Manager
- Circuit Breaker Pattern (Release It! - Michael Nygard)
