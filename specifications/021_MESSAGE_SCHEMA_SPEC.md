# 021 - Message Schema Specification

## Purpose

Define the schema for all messages exchanged on the Event Bus and IPC Protocol. Ensures type safety, validation, and backward compatibility.

## Scope

- Message envelope schema
- Payload schema definitions per topic
- Schema versioning
- Validation rules
- Evolution guidelines

## Responsibilities

- Define canonical message schemas
- Enforce schema validation on publish
- Document schema evolution rules
- Provide schema registry

## Non-Goals

- Serialization format (JSON is standard)
- Compression (transport layer concern)
- Encryption (security layer concern)

## Architecture Position

Message Schema is a foundational contract used by Event Bus, IPC Protocol, and all subsystems.

## Message Envelope Schema

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "required": ["id", "topic", "timestamp", "source", "payload"],
  "properties": {
    "id": {
      "type": "string",
      "format": "uuid",
      "description": "Unique message identifier"
    },
    "topic": {
      "type": "string",
      "pattern": "^[a-z][a-z0-9_]*\.[a-z][a-z0-9_]*\.[a-z][a-z0-9_]*$",
      "description": "Hierarchical topic path (subsystem.component.action)"
    },
    "timestamp": {
      "type": "string",
      "format": "date-time",
      "description": "ISO 8601 timestamp"
    },
    "source": {
      "type": "string",
      "pattern": "^[a-z][a-z0-9_]*$",
      "description": "Emitting subsystem name"
    },
    "payload": {
      "type": "object",
      "description": "Event-specific payload"
    },
    "correlation_id": {
      "type": ["string", "null"],
      "format": "uuid",
      "description": "Request/response correlation ID"
    },
    "priority": {
      "type": "string",
      "enum": ["normal", "high", "critical"],
      "default": "normal"
    }
  }
}
```

## Topic Registry

| Topic | Payload Schema | Description |
|-------|---------------|-------------|
| `kernel.initialized` | RuntimeHandle | Kernel ready |
| `kernel.shutdown_requested` | ShutdownMode | Shutdown signal |
| `runtime.state.changed` | StateTransition | Runtime state change |
| `audio.captured` | AudioFrame | Raw audio frame |
| `audio.vad.detected` | VADEvent | Voice activity detected |
| `audio.wake.detected` | WakeEvent | Wake word detected |
| `audio.stt.result` | STTResult | Speech transcription |
| `audio.tts.request` | TTSRequest | Text-to-speech request |
| `cognitive.context.assembled` | ContextWindow | Assembled context |
| `cognitive.plan.created` | Plan | Generated plan |
| `cognitive.tool.called` | ToolCall | Tool invocation |
| `memory.retrieved` | MemoryResult | Retrieved memories |
| `automation.action.executed` | ActionResult | UI action result |
| `vision.screen.captured` | ScreenFrame | Screen capture frame |
| `plugin.event.received` | PluginEvent | Plugin-generated event |

## Schema Evolution Rules

1. **Additive Changes Only**: New fields may be added; existing fields MUST NOT be removed
2. **Default Values**: New fields MUST have sensible defaults
3. **Type Stability**: Field types MUST NOT change
4. **Topic Stability**: Topic names are immutable
5. **Versioning**: Major schema changes require new topic

## Validation

- All messages MUST validate against envelope schema
- Payloads SHOULD validate against topic-specific schema
- Validation failures are logged and message is dropped

## Cross References

- [012_EVENT_BUS_SPEC.md](012_EVENT_BUS_SPEC.md)
- [020_IPC_PROTOCOL_SPEC.md](020_IPC_PROTOCOL_SPEC.md)
- [022_COMMAND_SCHEMA_SPEC.md](022_COMMAND_SCHEMA_SPEC.md)

## References

- JSON Schema Draft 2020-12
- Apache Avro Schema Evolution
- Protocol Buffers Versioning
