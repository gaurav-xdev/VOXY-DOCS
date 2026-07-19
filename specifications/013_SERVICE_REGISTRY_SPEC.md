# 013 - Service Registry Specification

## Purpose

Define the service discovery and registration system that maintains a directory of all active VOXY services, their capabilities, health status, and endpoint information.

## Scope

- Service registration and deregistration
- Capability advertisement and discovery
- Health status aggregation
- Service endpoint resolution
- Service metadata management

## Responsibilities

- Maintain authoritative directory of all services
- Enable dynamic service discovery
- Track service capabilities and versions
- Aggregate health status from Health Monitor
- Support service lookup by name, capability, or tag

## Non-Goals

- Service mesh networking (not needed for single-process architecture)
- Load balancing (handled by consumers)
- Service configuration management (Configuration subsystem)

## Architecture Position

The Service Registry sits alongside the Event Bus and Runtime Manager. It provides the "yellow pages" for VOXY services.

## Inputs

- Service registration requests from subsystems
- Health status updates from Health Monitor
- Deregistration events from Runtime Manager
- Query requests from consumers

## Outputs

- Service directory queries
- Service status broadcasts
- Capability match results

## Interfaces

### Public Contracts

```rust
pub struct ServiceRegistry {
    pub fn register(spec: ServiceSpec) -> Result<ServiceHandle, RegistryError>;
    pub fn deregister(handle: ServiceHandle);
    pub fn lookup(name: &str) -> Option<ServiceInfo>;
    pub fn query(capability: &str) -> Vec<ServiceInfo>;
    pub fn list_all() -> Vec<ServiceInfo>;
    pub fn update_health(handle: ServiceHandle, status: HealthStatus);
}
```

## Data Models

### ServiceSpec

| Field | Type | Description |
|-------|------|-------------|
| name | String | Unique service name |
| version | SemVer | Service version |
| capabilities | Vec<String> | Advertised capabilities |
| endpoints | Vec<Endpoint> | Communication endpoints |
| tags | Vec<String> | Searchable tags |
| metadata | Map | Additional service info |

### ServiceInfo

| Field | Type | Description |
|-------|------|-------------|
| spec | ServiceSpec | Original registration |
| health | HealthStatus | Current health |
| registered_at | DateTime | Registration timestamp |
| last_heartbeat | DateTime | Last health update |

## State Management

- Services are stored in-memory with no persistence
- Registry state is reconstructed on restart via re-registration
- Health status is ephemeral

## Threading Model

Registry operations are synchronized via `RwLock`. Reads are concurrent; writes are exclusive.

## IPC Requirements

- Registry is in-process only
- External processes query via IPC Protocol (020)

## Event Definitions

| Event | Payload | Description |
|-------|---------|-------------|
| `registry.service.registered` | ServiceInfo | New service registered |
| `registry.service.deregistered` | name | Service removed |
| `registry.service.health_changed` | (name, old, new) | Health status update |
| `registry.query` | (capability, results) | Capability query performed |

## Error Handling

### RegistryError Taxonomy

| Code | Description | Recovery |
|------|-------------|----------|
| `SR001` | Duplicate registration | Reject, log warning |
| `SR002` | Service not found | Return None, log debug |
| `SR003` | Invalid capability | Reject, log error |
| `SR004` | Registry full | Reject, alert |

## Retry Strategy

- Registration: 3 retries on conflict
- Health updates: Fire-and-forget

## Recovery Strategy

- Registry corruption: Rebuild from re-registrations
- Memory pressure: Evict stale entries (no heartbeat > 5min)

## Security Requirements

- Service names are namespaced by subsystem
- Only registered services can update their own health
- Read access is unrestricted within process

## Configuration

```json
{
  "service_registry": {
    "max_services": 1000,
    "heartbeat_timeout_seconds": 300,
    "enable_query_logging": false,
    "cache_ttl_seconds": 60
  }
}
```

## Performance Targets

| Metric | Target |
|--------|--------|
| Registration latency | < 1ms |
| Lookup latency | < 100us |
| Query latency | < 1ms |
| Health update latency | < 100us |

## Resource Limits

| Resource | Limit |
|----------|-------|
| Registered services | 1000 |
| Capabilities per service | 50 |
| Endpoints per service | 10 |

## Benchmarks

- `bench_registration`: Register 1000 services
- `bench_lookup`: 1M lookups
- `bench_query`: Complex capability queries

## Testing Requirements

- Concurrent registration/deregistration tests
- Health status propagation tests
- Query accuracy tests
- Memory leak tests

## Logging Requirements

- Registrations/deregistrations at INFO level
- Health changes at DEBUG level
- Query results at TRACE level

## Telemetry

- Service count by subsystem
- Registration/deregistration rate
- Query frequency by capability
- Health status distribution

## Cross References

- [011_RUNTIME_MANAGER_SPEC.md](011_RUNTIME_MANAGER_SPEC.md)
- [012_EVENT_BUS_SPEC.md](012_EVENT_BUS_SPEC.md)
- [017_HEALTH_MONITOR_SPEC.md](017_HEALTH_MONITOR_SPEC.md)

## References

- Consul Service Discovery
- etcd Registry Pattern
- Windows Service Control Manager
