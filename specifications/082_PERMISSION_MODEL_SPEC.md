# 082 - Permission Model Specification

## Purpose

Define the capability-based permission model that controls what actions plugins, tools, and subsystems can perform within VOXY.

## Scope

- Capability token system
- Permission declarations
- Runtime permission enforcement
- Permission revocation
- Audit logging
- User consent management

## Responsibilities

- Define capability taxonomy
- Issue and validate capability tokens
- Enforce permissions at runtime
- Support permission revocation
- Log permission events
- Manage user consent

## Non-Goals

- Authentication (Security Model)
- Encryption (Secret Storage)
- Network security

## Architecture Position

Permission Model is a cross-cutting security layer used by Plugin Runtime, Tool Calling, and Automation Runtime.

## Capability Taxonomy

| Capability | Description | Risk Level |
|------------|-------------|------------|
| `audio:capture` | Capture microphone audio | High |
| `audio:loopback` | Capture system audio | Medium |
| `automation:read` | Read UI state | Low |
| `automation:write` | Interact with UI | High |
| `automation:admin` | Admin-level automation | Critical |
| `file:read` | Read files | Medium |
| `file:write` | Write files | High |
| `file:delete` | Delete files | Critical |
| `memory:read` | Read memory | Low |
| `memory:write` | Write memory | Medium |
| `network:read` | HTTP GET requests | Medium |
| `network:write` | HTTP POST/PUT | High |
| `process:spawn` | Spawn processes | Critical |
| `registry:read` | Read registry | Medium |
| `registry:write` | Write registry | Critical |
| `screen:capture` | Capture screen | High |
| `system:info` | Read system info | Low |

## Data Models

### CapabilityToken

| Field | Type | Description |
|-------|------|-------------|
| id | UUID | Token identifier |
| subject | String | Token holder |
| capabilities | Vec<String> | Granted capabilities |
| issued_at | DateTime | Issue time |
| expires_at | Option<DateTime> | Expiration |
| signature | String | Token signature |

### PermissionGrant

| Field | Type | Description |
|-------|------|-------------|
| capability | String | Capability |
| granted | bool | Is granted |
| granted_at | DateTime | Grant time |
| granted_by | String | Grantor |
| expires_at | Option<DateTime> | Expiration |

## Interfaces

```rust
pub struct PermissionModel {
    pub fn check(subject: &str, capability: &str) -> Result<(), PermissionError>;
    pub fn grant(subject: &str, capability: &str, grantor: &str) -> Result<CapabilityToken, PermissionError>;
    pub fn revoke(token: &CapabilityToken);
    pub fn list_grants(subject: &str) -> Vec<PermissionGrant>;
}
```

## State Management

- Grants stored in SQLite
- Tokens validated in-memory
- Audit log persisted

## Threading Model

Permission checks are lock-free (read-only). Grants require synchronization.

## IPC Requirements

- Token validation via IPC for sandboxed plugins

## Event Definitions

| Event | Payload | Description |
|-------|---------|-------------|
| `permission.granted` | PermissionGrant | Permission granted |
| `permission.revoked` | (subject, capability) | Permission revoked |
| `permission.denied` | (subject, capability) | Access denied |
| `permission.audit` | AuditEntry | Audit event |

## Error Handling

| Code | Description | Recovery |
|------|-------------|----------|
| `PM001` | Permission denied | Request grant |
| `PM002` | Token expired | Reissue |
| `PM003` | Invalid token | Reject |

## Retry Strategy

- None for permission checks

## Recovery Strategy

- Denied: Request user consent
- Expired: Reissue

## Security Requirements

- Tokens are signed with HMAC-SHA256
- Token validation is constant-time
- Audit log is append-only
- Grants require user confirmation for High/Critical

## Configuration

```json
{
  "permission_model": {
    "token_ttl_hours": 168,
    "require_user_consent_for": ["High", "Critical"],
    "audit_log_retention_days": 90,
    "max_grants_per_subject": 50
  }
}
```

## Performance Targets

| Metric | Target |
|--------|--------|
| Check latency | < 1ms |
| Grant latency | < 10ms |

## Resource Limits

| Resource | Limit |
|----------|-------|
| Active tokens | 10,000 |
| Grants per subject | 50 |
| Audit log | 1M entries |

## Benchmarks

- `bench_check`: Permission check speed
- `bench_grant`: Grant speed

## Testing Requirements

- Permission enforcement tests
- Token validation tests
- Revocation tests
- Audit tests

## Logging Requirements

- Grants at INFO level
- Denials at WARN level
- Audit at INFO level

## Telemetry

- Grant frequency
- Denial rate
- Token usage

## Cross References

- [080_PLUGIN_RUNTIME_SPEC.md](080_PLUGIN_RUNTIME_SPEC.md)
- [090_SECURITY_MODEL_SPEC.md](090_SECURITY_MODEL_SPEC.md)
- [091_POLICY_ENGINE_SPEC.md](091_POLICY_ENGINE_SPEC.md)

## References

- Capability-Based Security
- macOS Entitlements
- Android Permissions
