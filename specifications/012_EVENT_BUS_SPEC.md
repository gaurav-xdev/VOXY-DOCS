# 012 - Event Bus Specification

## Purpose

Define the central message distribution system that enables decoupled, asynchronous communication between all VOXY subsystems. The Event Bus is the nervous system of VOXY.

## Scope

- Message routing and delivery
- Topic-based publish/subscribe
- Message serialization and schema validation
- Delivery guarantees (at-least-once)
- Backpressure and flow control
- Message ordering within topics

## Responsibilities

- Route events from publishers to subscribers
- Enforce message schema validation
- Provide delivery guarantees
- Handle backpressure when consumers are slow
- Maintain message ordering per topic partition
- Support event replay for recovery

## Non-Goals

- Persistent message queue (use IPC Protocol for durable messages)
- Cross-machine messaging (single-process only)
- Exactly-once semantics (at-least-once is sufficient)

## Architecture Position

The Event Bus is a core infrastructure service used by all subsystems. It sits between publishers and subscribers, mediating all inter-subsystem communication.

## Inputs

- Published events from any subsystem
- Subscription requests from consumers
- Schema definitions from Configuration

## Outputs

- Delivered events to subscribers
- Delivery confirmation (optional)
- Dead letter events (undeliverable)

## Interfaces

### Public Contracts

```rust
pub struct EventBus {
    pub fn publish(topic: &str, event: Event) -> Result<(), BusError>;
    pub fn subscribe(topic: &str, handler: EventHandler) -> Subscription;
    pub fn unsubscribe(sub: Subscription);
    pub fn request_reply(topic: &str, event: Event, timeout: Duration) -> Result<Event, BusError>;
}
```

### Internal Contracts

- Topics are hierarchical: `subsystem.component.action`
- Wildcards supported: `audio.*.started`, `*.capture.*`
- Message size limit: 1MB
- Queue depth per subscriber: 1000 messages

## Data Models

### Event

| Field | Type | Description |
|-------|------|-------------|
| id | UUID | Unique event identifier |
| topic | String | Hierarchical topic path |
| timestamp | DateTime<Utc> | Event creation time |
| source | String | Emitting subsystem name |
| payload | JSON | Event-specific data |
| correlation_id | Option<UUID> | Request/response correlation |
| priority | Priority | Normal, High, Critical |

### Subscription

| Field | Type | Description |
|-------|------|-------------|
| id | UUID | Subscription identifier |
| topic_pattern | String | Subscribed topic pattern |
| handler | EventHandler | Callback function |
| max_inflight | usize | Max concurrent messages |

## State Management

- Topics are created lazily on first publish
- Subscriptions are active immediately upon creation
- No persistent state; all state is in-memory

## Threading Model

Event Bus runs on the Tokio runtime. Publishers are non-blocking. Each subscriber has a dedicated async task that processes messages sequentially per topic partition.

## IPC Requirements

- Event Bus is in-process only
- For cross-process communication, use IPC Protocol (020)
- Event Bus MAY forward select events to IPC for external consumers

## Event Definitions

| Event | Payload | Description |
|-------|---------|-------------|
| `bus.subscriber.lagged` | (topic, sub_id, count) | Subscriber fell behind |
| `bus.message.dropped` | (topic, reason) | Message dropped due to backpressure |
| `bus.schema.violation` | (topic, error) | Message failed schema validation |

## Error Handling

### BusError Taxonomy

| Code | Description | Recovery |
|------|-------------|----------|
| `EB001` | Topic not found | Auto-create, log info |
| `EB002` | Schema validation failed | Drop message, log error |
| `EB003` | Subscriber queue full | Drop oldest, log warning |
| `EB004` | Request timeout | Return timeout error |
| `EB005` | Serialization failed | Drop message, log error |

## Retry Strategy

- Message delivery: 3 attempts with 100ms backoff
- Subscriber lag: Reduce fetch rate, alert monitoring

## Recovery Strategy

- Subscriber crash: Remove subscription, notify Health Monitor
- Bus overload: Apply backpressure, shed non-critical messages

## Security Requirements

- Events carry source attribution
- Sensitive topics require capability tokens
- Message payloads are not encrypted in-memory (process boundary is the trust boundary)

## Configuration

```json
{
  "event_bus": {
    "max_message_size_bytes": 1048576,
    "max_queue_depth": 1000,
    "subscriber_timeout_ms": 5000,
    "enable_schema_validation": true,
    "backpressure_strategy": "drop_oldest",
    "topic_partitions": 4
  }
}
```

## Performance Targets

| Metric | Target |
|--------|--------|
| Publish latency (p99) | < 1ms |
| End-to-end delivery (p99) | < 5ms |
| Throughput | > 100,000 events/sec |
| Subscriber lag detection | < 100ms |

## Resource Limits

| Resource | Limit |
|----------|-------|
| Active topics | 10,000 |
| Subscriptions per topic | 100 |
| In-flight messages | 50,000 |
| Message retention | 0 (fire-and-forget) |

## Benchmarks

- `bench_publish_latency`: Single publisher, single subscriber
- `bench_throughput`: Max events per second
- `bench_fanout`: Single publisher, 100 subscribers

## Testing Requirements

- Property tests for ordering guarantees
- Backpressure stress tests
- Schema validation tests
- Subscriber crash recovery tests

## Logging Requirements

- Dropped messages at WARN level
- Schema violations at ERROR level
- Subscriber lag at WARN level
- Normal operation at TRACE level

## Telemetry

- Messages published per topic
- Messages delivered per subscriber
- Queue depth per subscriber
- Delivery latency histogram
- Drop rate

## Cross References

- [010_KERNEL_SPEC.md](010_KERNEL_SPEC.md)
- [020_IPC_PROTOCOL_SPEC.md](020_IPC_PROTOCOL_SPEC.md)
- [021_MESSAGE_SCHEMA_SPEC.md](021_MESSAGE_SCHEMA_SPEC.md)
- [110_CONFIGURATION_SCHEMA.md](110_CONFIGURATION_SCHEMA.md)

## References

- Tokio Broadcast Channels
- NATS Messaging Protocol
- Apache Kafka Topic Design
