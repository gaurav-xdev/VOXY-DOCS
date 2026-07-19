# 010 - Kernel Specification

## Purpose

Define the foundational runtime kernel responsible for process lifecycle management, thread pool allocation, resource governance, and subsystem initialization. The Kernel is the root of all VOXY runtime operations.

## Scope

- Process initialization and shutdown orchestration
- Thread pool management and work scheduling
- Memory allocation tracking and limits
- Subsystem bootstrap and dependency resolution
- Signal handling and crash recovery
- Resource quota enforcement

## Responsibilities

- Initialize the VOXY runtime in correct dependency order
- Manage the lifecycle of all subsystems
- Provide a unified thread pool for CPU-bound work
- Enforce resource limits (memory, CPU, handles)
- Coordinate graceful and emergency shutdown
- Collect and report runtime diagnostics

## Non-Goals

- Subsystem-specific logic (delegated to respective managers)
- User interface rendering
- Network I/O management (handled by Tokio runtime)
- AI model inference (handled by Model Router)

## Architecture Position

The Kernel sits at the base of the VOXY architecture. All other subsystems register with the Kernel during bootstrap. The Kernel owns the Tokio runtime instance and distributes `tokio::runtime::Handle` to subsystems.

## Inputs

- Configuration manifest (`voxy.config.json`)
- Command-line arguments
- Windows environment variables
- Registry settings (read-only, HKLM and HKCU)

## Outputs

- Initialized subsystem registry
- Runtime handles (thread pool, event loop)
- Health status broadcasts
- Diagnostic telemetry

## Interfaces

### Public Contracts

```rust
// Conceptual interface - specification only
pub struct Kernel {
    pub fn initialize(config: Configuration) -> Result<RuntimeHandle, KernelError>;
    pub fn shutdown(mode: ShutdownMode) -> Result<(), KernelError>;
    pub fn register_subsystem(spec: SubsystemSpec) -> Result<SubsystemHandle, KernelError>;
    pub fn get_runtime_handle() -> tokio::runtime::Handle;
    pub fn enforce_quota(resource: ResourceType, limit: usize) -> Result<(), QuotaError>;
}
```

### Internal Contracts

- Subsystem initialization MUST complete within 30 seconds or trigger recovery
- Thread pool size defaults to `num_cpus * 2`, configurable via `kernel.thread_pool_size`
- Memory quota defaults to 2GB, enforced via custom allocator hooks

## Data Models

### SubsystemSpec

| Field | Type | Description |
|-------|------|-------------|
| name | String | Unique subsystem identifier |
| version | SemVer | Subsystem version |
| dependencies | Vec<String> | Ordered list of dependency subsystem names |
| init_timeout_ms | u64 | Maximum initialization time |
| resource_claims | ResourceClaims | Memory, CPU, handle requirements |

### RuntimeHandle

| Field | Type | Description |
|-------|------|-------------|
| thread_pool | tokio::runtime::Handle | Async runtime handle |
| shutdown_tx | broadcast::Sender | Shutdown signal channel |
| health_tx | mpsc::Sender | Health report channel |

## State Management

### Kernel States

```
[Uninitialized] -> [Bootstrapping] -> [Running] -> [ShuttingDown] -> [Stopped]
                      |                                      |
                      +-> [Degraded] <-----------------------+
```

- **Uninitialized**: Process started, no subsystems loaded
- **Bootstrapping**: Subsystems initializing in dependency order
- **Running**: All critical subsystems healthy
- **Degraded**: One or more non-critical subsystems failed
- **ShuttingDown**: Graceful shutdown in progress
- **Stopped**: All subsystems terminated, process exit

## Threading Model

The Kernel spawns:

1. **Main Thread**: Event loop for system signals and lifecycle events
2. **Tokio Worker Threads**: `num_cpus * 2` threads for async task execution
3. **Blocking Pool**: Unbounded (but monitored) for blocking I/O
4. **Monitor Thread**: Periodic health checks and resource accounting

All subsystem code MUST use the Tokio runtime. Direct OS thread creation is prohibited without Kernel approval.

## IPC Requirements

The Kernel does not directly participate in IPC. It provides the runtime infrastructure that IPC subsystems (020) use.

## Event Definitions

| Event | Payload | Emitter | Consumers |
|-------|---------|---------|-----------|
| `kernel.initialized` | RuntimeHandle | Kernel | All Subsystems |
| `kernel.shutdown_requested` | ShutdownMode | Signal Handler | All Subsystems |
| `kernel.subsystem_ready` | SubsystemName | Subsystem | Kernel, Health Monitor |
| `kernel.subsystem_failed` | (Name, Error) | Subsystem | Kernel, Health Monitor, Event Bus |
| `kernel.resource_warning` | ResourceAlert | Monitor | Logging, Metrics |

## Error Handling

### KernelError Taxonomy

| Code | Description | Recovery |
|------|-------------|----------|
| `K001` | Configuration parse failure | Fatal - exit process |
| `K002` | Subsystem initialization timeout | Retry once, then degraded |
| `K003` | Circular dependency detected | Fatal - exit process |
| `K004` | Resource quota exceeded | Throttle, alert, continue |
| `K005` | Memory allocation failure | Emergency shutdown |
| `K006` | Thread pool exhaustion | Backpressure, queue growth alert |

## Retry Strategy

- Subsystem initialization: 1 retry with exponential backoff (1s, 2s)
- Resource allocation: Immediate failure, no retry
- Configuration reload: 3 retries with 500ms intervals

## Recovery Strategy

- **Graceful Shutdown**: Notify all subsystems, wait for cleanup (max 30s)
- **Emergency Shutdown**: Abort all threads, dump state, exit
- **Degraded Mode**: Continue with failed non-critical subsystems disabled

## Security Requirements

- Process runs at standard user integrity level
- No elevation required for normal operation
- Configuration files MUST be validated against schema before parsing
- Environment variable injection is blocked for sensitive keys

## Configuration

```json
{
  "kernel": {
    "thread_pool_size": 16,
    "memory_quota_mb": 2048,
    "init_timeout_seconds": 30,
    "shutdown_timeout_seconds": 30,
    "enable_telemetry": true,
    "log_level": "INFO",
    "crash_reporting": true
  }
}
```

## Performance Targets

| Metric | Target |
|--------|--------|
| Cold start time | < 3 seconds |
| Subsystem init per subsystem | < 2 seconds |
| Shutdown time | < 5 seconds |
| Thread pool latency (p99) | < 10ms |

## Resource Limits

| Resource | Limit | Action on Exceed |
|----------|-------|------------------|
| RAM | 2GB | Log warning, trigger GC |
| CPU | 50% (single core) | Throttle background tasks |
| Handles | 10,000 | Log warning |
| Threads | 256 | Block creation, alert |

## Benchmarks

- `bench_kernel_startup`: Measure cold start to Ready state
- `bench_subsystem_init`: Measure per-subsystem initialization time
- `bench_thread_pool`: Measure task scheduling latency under load

## Testing Requirements

- Unit tests for dependency resolution algorithm
- Integration tests for full bootstrap/shutdown cycle
- Chaos tests for subsystem failure injection
- Resource exhaustion tests

## Logging Requirements

- Structured JSON logging via `tracing` crate
- Log level: INFO for normal operation, DEBUG for diagnostics
- All state transitions MUST be logged
- Resource warnings MUST be logged at WARN level

## Telemetry

- Startup time histogram
- Subsystem initialization duration per subsystem
- Thread pool queue depth
- Memory allocation rate
- Active handle count

## Cross References

- [011_RUNTIME_MANAGER_SPEC.md](011_RUNTIME_MANAGER_SPEC.md)
- [012_EVENT_BUS_SPEC.md](012_EVENT_BUS_SPEC.md)
- [013_SERVICE_REGISTRY_SPEC.md](013_SERVICE_REGISTRY_SPEC.md)
- [017_HEALTH_MONITOR_SPEC.md](017_HEALTH_MONITOR_SPEC.md)
- [090_SECURITY_MODEL_SPEC.md](090_SECURITY_MODEL_SPEC.md)

## References

- Tokio Runtime Documentation
- Windows Process Lifecycle
- Rust `std::alloc` System Allocator
