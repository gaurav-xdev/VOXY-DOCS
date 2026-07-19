# 080 - Plugin Runtime Specification

## Purpose

Define the Plugin Runtime subsystem that manages the lifecycle, execution, and isolation of third-party plugins that extend VOXY's capabilities.

## Scope

- Plugin loading and unloading
- Plugin lifecycle management
- Plugin event routing
- Plugin capability enforcement
- Plugin sandbox coordination
- Plugin marketplace integration

## Responsibilities

- Load and validate plugins
- Manage plugin lifecycle
- Route events to plugins
- Enforce plugin permissions
- Coordinate with sandbox
- Handle plugin errors

## Non-Goals

- Plugin development tools (SDK)
- Plugin store/marketplace UI
- Plugin review process

## Architecture Position

Plugin Runtime sits above Plugin SDK and Sandbox, managing all plugin operations.

## Inputs

- Plugin packages
- User installation requests
- Event bus messages
- Configuration

## Outputs

- Plugin events
- Plugin status
- Error reports

## Interfaces

```rust
pub struct PluginRuntime {
    pub fn load_plugin(path: &Path) -> Result<PluginHandle, PluginError>;
    pub fn unload_plugin(handle: PluginHandle);
    pub fn list_plugins() -> Vec<PluginInfo>;
    pub fn send_event(handle: PluginHandle, event: PluginEvent);
    pub fn get_plugin_state(handle: PluginHandle) -> PluginState;
}
```

## Data Models

### PluginInfo

| Field | Type | Description |
|-------|------|-------------|
| id | String | Plugin identifier |
| name | String | Plugin name |
| version | SemVer | Plugin version |
| author | String | Plugin author |
| capabilities | Vec<String> | Required capabilities |
| permissions | Vec<String> | Granted permissions |
| state | PluginState | Current state |

### PluginState

| Variant | Description |
|---------|-------------|
| `Installed` | Installed but not loaded |
| `Loading` | Loading in progress |
| `Running` | Active and operational |
| `Paused` | Temporarily suspended |
| `Error` | Error state |
| `Unloaded` | Not loaded |

## State Management

- Plugin states tracked in-memory
- Configuration persisted

## Threading Model

Plugins run in sandboxed processes. Communication via IPC.

## IPC Requirements

- All plugin communication via IPC Protocol
- Sandboxed process isolation

## Event Definitions

| Event | Payload | Description |
|-------|---------|-------------|
| `plugin.loaded` | PluginInfo | Plugin loaded |
| `plugin.unloaded` | id | Plugin unloaded |
| `plugin.error` | (id, error) | Plugin error |
| `plugin.event_received` | (id, event) | Event from plugin |

## Error Handling

| Code | Description | Recovery |
|------|-------------|----------|
| `PR001` | Plugin load failure | Reject, alert user |
| `PR002` | Plugin crash | Restart, then disable |
| `PR003` | Permission violation | Block, alert |
| `PR004` | IPC failure | Reconnect, retry |

## Retry Strategy

- Load: 1 retry
- IPC: 3 retries
- Crash: Restart once

## Recovery Strategy

- Crash: Restart once, then disable
- IPC failure: Reconnect
- Permission violation: Disable

## Security Requirements

- Plugins run in sandbox
- Capabilities enforced
- Code signing verified
- No elevation

## Configuration

```json
{
  "plugin_runtime": {
    "plugin_directory": "%LOCALAPPDATA%/VOXY/plugins/",
    "max_plugins": 50,
    "sandbox_enabled": true,
    "auto_restart": true,
    "max_restart_count": 3,
    "code_signing_required": true
  }
}
```

## Performance Targets

| Metric | Target |
|--------|--------|
| Load time | < 2s |
| Event latency | < 10ms |
| Memory per plugin | < 100MB |

## Resource Limits

| Resource | Limit |
|----------|-------|
| Max plugins | 50 |
| Memory per plugin | 100MB |
| IPC message size | 1MB |

## Benchmarks

- `bench_load`: Plugin load time
- `bench_event`: Event routing

## Testing Requirements

- Load/unload tests
- Crash recovery tests
- Permission tests
- IPC tests

## Logging Requirements

- Load/unload at INFO level
- Errors at ERROR level
- Events at DEBUG level

## Telemetry

- Plugin count
- Load/unload frequency
- Error rate
- Event volume

## Cross References

- [081_PLUGIN_SDK_SPEC.md](081_PLUGIN_SDK_SPEC.md)
- [082_PERMISSION_MODEL_SPEC.md](082_PERMISSION_MODEL_SPEC.md)
- [083_SANDBOX_SPEC.md](083_SANDBOX_SPEC.md)
- [020_IPC_PROTOCOL_SPEC.md](020_IPC_PROTOCOL_SPEC.md)

## References

- Windows Sandbox
- Chrome Extension Model
- VS Code Extension API
