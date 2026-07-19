# 092 - Secret Storage Specification

## Purpose

Define the Secret Storage subsystem that securely stores API keys, passwords, tokens, and other sensitive credentials using Windows Data Protection API (DPAPI) and hardware-backed encryption where available.

## Scope

- Credential storage
- Encryption at rest
- Secure retrieval
- Key rotation
- Backup and recovery
- Audit logging

## Responsibilities

- Store secrets encrypted
- Retrieve secrets on demand
- Support key rotation
- Backup secrets securely
- Log access
- Integrate with Windows Credential Manager

## Non-Goals

- Password generation
- Secret sharing
- External secret management (HashiCorp Vault, etc.)

## Architecture Position

Secret Storage is a foundational security service used by Model Router, Plugin Runtime, and Configuration.

## Data Models

### Secret

| Field | Type | Description |
|-------|------|-------------|
| id | String | Secret identifier |
| type | SecretType | APIKey, Password, Token, etc. |
| encrypted_value | Vec<u8> | Encrypted data |
| metadata | Map | Non-sensitive metadata |
| created_at | DateTime | Creation time |
| last_rotated | DateTime | Last rotation |
| access_count | u32 | Access frequency |

### SecretType

| Variant | Description |
|---------|-------------|
| `APIKey` | API key |
| `Password` | Password |
| `Token` | Bearer token |
| `Certificate` | TLS certificate |
| `ConnectionString` | Database connection |

## Interfaces

```rust
pub struct SecretStorage {
    pub fn store(id: &str, secret: &str, secret_type: SecretType) -> Result<(), SecretError>;
    pub fn retrieve(id: &str) -> Result<String, SecretError>;
    pub fn delete(id: &str) -> Result<(), SecretError>;
    pub fn rotate(id: &str) -> Result<(), SecretError>;
    pub fn list() -> Vec<SecretInfo>;
}
```

## Encryption

- Primary: Windows DPAPI (user-scoped)
- Secondary: AES-256-GCM with hardware-backed key
- Key derived from Windows Hello / TPM where available

## State Management

- Secrets stored in SQLite with encryption
- Master key protected by DPAPI
- No plaintext secrets in memory longer than necessary

## Threading Model

Secret operations run on blocking pool. Memory is zeroed after use.

## IPC Requirements

- No IPC for secret storage
- External processes cannot access secrets

## Event Definitions

| Event | Payload | Description |
|-------|---------|-------------|
| `secret.stored` | id | Secret stored |
| `secret.retrieved` | id | Secret accessed |
| `secret.deleted` | id | Secret deleted |
| `secret.rotated` | id | Secret rotated |

## Error Handling

| Code | Description | Recovery |
|------|-------------|----------|
| `SS001` | Secret not found | Return error |
| `SS002` | Decryption failed | Alert, manual recovery |
| `SS003` | Storage full | Alert, cleanup |

## Retry Strategy

- None

## Recovery Strategy

- Decryption failure: Alert user, manual recovery
- Storage full: Archive old secrets

## Security Requirements

- Secrets never logged
- Memory zeroed after use
- DPAPI encryption mandatory
- Access audited
- No plaintext backup

## Configuration

```json
{
  "secret_storage": {
    "encryption": "dpapi",
    "backup_enabled": true,
    "backup_location": "%LOCALAPPDATA%/VOXY/secrets/backup/",
    "rotation_interval_days": 90,
    "max_secrets": 1000,
    "audit_access": true
  }
}
```

## Performance Targets

| Metric | Target |
|--------|--------|
| Store latency | < 50ms |
| Retrieve latency | < 20ms |
| Memory overhead | < 1MB |

## Resource Limits

| Resource | Limit |
|----------|-------|
| Max secrets | 1000 |
| Secret size | 10KB |
| Memory cache | 10 secrets |

## Benchmarks

- `bench_store`: Storage speed
- `bench_retrieve`: Retrieval speed

## Testing Requirements

- Encryption/decryption tests
- DPAPI integration tests
- Rotation tests
- Audit tests

## Logging Requirements

- Access at INFO level (no values)
- Errors at ERROR level
- Rotation at INFO level

## Telemetry

- Access frequency
- Storage size
- Rotation events

## Cross References

- [090_SECURITY_MODEL_SPEC.md](090_SECURITY_MODEL_SPEC.md)
- [014_CONFIGURATION_SPEC.md](014_CONFIGURATION_SPEC.md)

## References

- Windows DPAPI
- Windows Credential Manager
- AES-256-GCM
- TPM 2.0
