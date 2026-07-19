# 111 - Error Codes Specification

## Purpose

Define the complete error code taxonomy for VOXY, ensuring consistent error handling, debugging, and user communication across all subsystems.

## Scope

- Error code taxonomy
- Error code ranges per subsystem
- Error severity levels
- Error message templates
- Error recovery guidance

## Responsibilities

- Define canonical error codes
- Ensure uniqueness
- Document recovery actions
- Support localization
- Enable automated handling

## Error Code Format

```
[SS][NNN]
```

- `SS`: Two-letter subsystem code
- `NNN`: Three-digit sequential number

## Subsystem Codes

| Code | Subsystem |
|------|-----------|
| `K` | Kernel |
| `RM` | Runtime Manager |
| `EB` | Event Bus |
| `SR` | Service Registry |
| `CF` | Configuration |
| `LG` | Logging |
| `MT` | Metrics |
| `HM` | Health Monitor |
| `IP` | IPC Protocol |
| `AP` | Audio Pipeline |
| `AC` | Audio Capture |
| `WW` | Wake Word |
| `VD` | VAD |
| `ST` | STT |
| `TT` | TTS |
| `BI` | Barge-In |
| `CA` | Context Assembly |
| `MR` | Model Router |
| `PL` | Planner |
| `EX` | Executor |
| `RF` | Reflection |
| `TC` | Tool Calling |
| `ME` | Memory Runtime |
| `WM` | Working Memory |
| `EM` | Episodic Memory |
| `SM` | Semantic Memory |
| `PM` | Procedural Memory |
| `MC` | Memory Consolidation |
| `RE` | Memory Retrieval |
| `AR` | Automation Runtime |
| `GE` | Grounding Engine |
| `UA` | UI Automation |
| `AE` | Action Executor |
| `SC` | Screen Capture |
| `OC` | OCR |
| `VR` | Vision Runtime |
| `OD` | Object Detection |
| `UA` | UI Analysis |
| `PR` | Plugin Runtime |
| `SE` | Security |
| `PE` | Policy Engine |
| `SS` | Secret Storage |

## Severity Levels

| Level | Description | Action |
|-------|-------------|--------|
| `Critical` | System failure | Alert, shutdown |
| `Error` | Subsystem failure | Alert, retry |
| `Warning` | Degraded operation | Log, monitor |
| `Info` | Informational | Log |
| `Debug` | Debug info | Log (debug only) |

## Error Code Registry

### Kernel (K001-K099)

| Code | Description | Severity | Recovery |
|------|-------------|----------|----------|
| `K001` | Configuration parse failure | Critical | Exit |
| `K002` | Subsystem initialization timeout | Error | Retry, degrade |
| `K003` | Circular dependency detected | Critical | Exit |
| `K004` | Resource quota exceeded | Warning | Throttle |
| `K005` | Memory allocation failure | Critical | Shutdown |
| `K006` | Thread pool exhaustion | Error | Backpressure |

### Runtime Manager (RM001-RM099)

| Code | Description | Severity | Recovery |
|------|-------------|----------|----------|
| `RM001` | Subsystem not found | Error | Reject |
| `RM002` | Dependency not satisfied | Warning | Queue |
| `RM003` | Start timeout | Error | Mark failed |
| `RM004` | Stop timeout | Warning | Force stop |
| `RM005` | Invalid state transition | Error | Reject |
| `RM006` | Config apply failed | Error | Rollback |

### Event Bus (EB001-EB099)

| Code | Description | Severity | Recovery |
|------|-------------|----------|----------|
| `EB001` | Topic not found | Info | Auto-create |
| `EB002` | Schema validation failed | Error | Drop |
| `EB003` | Subscriber queue full | Warning | Drop oldest |
| `EB004` | Request timeout | Error | Return timeout |
| `EB005` | Serialization failed | Error | Drop |

### Memory (ME001-ME099)

| Code | Description | Severity | Recovery |
|------|-------------|----------|----------|
| `ME001` | Storage full | Error | Alert, stop |
| `ME002` | Embedding failure | Warning | Store without |
| `ME003` | Query timeout | Warning | Return partial |
| `ME004` | Database locked | Warning | Retry |
| `ME005` | Corruption detected | Critical | Restore |

### Model Router (MR001-MR099)

| Code | Description | Severity | Recovery |
|------|-------------|----------|----------|
| `MR001` | Provider timeout | Error | Fallback |
| `MR002` | Rate limit | Warning | Backoff |
| `MR003` | Invalid model | Error | Use default |
| `MR004` | Context too large | Error | Truncate |
| `MR005` | All providers failed | Critical | Return error |

### Automation (AR001-AR099)

| Code | Description | Severity | Recovery |
|------|-------------|----------|----------|
| `AR001` | Element not found | Warning | Retry |
| `AR002` | Action timeout | Error | Cancel |
| `AR003` | Window closed | Warning | Refresh |
| `AR004` | Permission denied | Error | Skip, notify |
| `AR005` | Automation API error | Error | Retry |

### Security (SE001-SE099)

| Code | Description | Severity | Recovery |
|------|-------------|----------|----------|
| `SE001` | Authentication failed | Warning | Retry, lockout |
| `SE002` | Authorization failed | Warning | Request grant |
| `SE003` | Policy violation | Error | Block, alert |
| `SE004` | Token expired | Warning | Reissue |
| `SE005` | Invalid token | Error | Reject |

## Error Response Format

```json
{
  "error": {
    "code": "MR001",
    "message": "Provider timeout: OpenAI API did not respond within 5000ms",
    "severity": "Error",
    "subsystem": "Model Router",
    "recovery": "Fallback to next provider in chain",
    "details": {
      "provider": "OpenAI",
      "timeout_ms": 5000,
      "fallback": "Azure"
    }
  }
}
```

## Cross References

- All subsystem specifications
- [015_LOGGING_SPEC.md](015_LOGGING_SPEC.md)
- [112_TELEMETRY_SPEC.md](112_TELEMETRY_SPEC.md)

## References

- HTTP Status Codes
- Windows Error Codes
- Rust Error Handling
