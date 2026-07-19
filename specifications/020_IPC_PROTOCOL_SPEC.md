# 020 - IPC Protocol Specification

## Purpose

Define the inter-process communication protocol that enables secure, efficient communication between VOXY core process, sandboxed plugins, and external tools.

## Scope

- Named pipe transport
- Message framing and serialization
- Authentication and capability verification
- Request/response and publish/subscribe patterns
- Connection lifecycle management

## Responsibilities

- Provide secure IPC channels between processes
- Authenticate connecting peers
- Enforce capability-based access control
- Handle connection failures and reconnection
- Support both sync and async communication patterns

## Non-Goals

- Network communication (local machine only)
- Cross-platform IPC (Windows-specific)
- High-throughput streaming (use shared memory)

## Architecture Position

IPC Protocol sits below the Event Bus for cross-process scenarios. It is used by Plugin Sandbox and external tooling.

## Inputs

- Connection requests from peers
- Messages from Event Bus (for forwarding)
- Capability tokens from Security Model

## Outputs

- Delivered messages to peers
- Connection status events
- Authentication results

## Interfaces

```rust
pub struct IpcProtocol {
    pub fn listen(endpoint: &str, auth: AuthConfig) -> Result<Listener, IpcError>;
    pub fn connect(endpoint: &str, token: CapabilityToken) -> Result<Connection, IpcError>;
    pub fn send(conn: &Connection, message: IpcMessage) -> Result<(), IpcError>;
    pub fn receive(conn: &Connection) -> Result<IpcMessage, IpcError>;
    pub fn close(conn: Connection);
}
```

## Data Models

### IpcMessage

| Field | Type | Description |
|-------|------|-------------|
| id | UUID | Message identifier |
| type | MessageType | Request, Response, Event, Command |
| payload | Vec<u8> | Serialized payload |
| timestamp | DateTime | Creation time |
| source | String | Source process ID |
| destination | Option<String> | Target process ID (None for broadcast) |
| correlation_id | Option<UUID> | Request/response correlation |

### Connection

| Field | Type | Description |
|-------|------|-------------|
| id | UUID | Connection identifier |
| peer_pid | u32 | Peer process ID |
| peer_name | String | Peer process name |
| capabilities | Vec<String> | Granted capabilities |
| established_at | DateTime | Connection time |
| last_activity | DateTime | Last message time |

## State Management

- Connections are tracked per peer
- Unauthenticated connections are dropped after 5 seconds
- Message queues are maintained per connection

## Threading Model

IPC uses async I/O on Tokio. Each connection has a dedicated read/write task pair.

## IPC Requirements

- Transport: Windows Named Pipes
- Framing: Length-prefixed protobuf messages
- Authentication: Capability tokens + process identity verification
- Encryption: Not required (same-machine, OS-enforced isolation)

## Event Definitions

| Event | Payload | Description |
|-------|---------|-------------|
| `ipc.peer.connected` | Connection | Peer connected |
| `ipc.peer.disconnected` | (id, reason) | Peer disconnected |
| `ipc.auth.failed` | (peer, reason) | Authentication failure |
| `ipc.message.received` | (conn_id, msg) | Message received |

## Error Handling

| Code | Description | Recovery |
|------|-------------|----------|
| `IP001` | Connection refused | Retry with backoff |
| `IP002` | Authentication failed | Reject, log security event |
| `IP003` | Message too large | Reject, close connection |
| `IP004` | Timeout | Retry once, then fail |
| `IP005` | Pipe broken | Reconnect |

## Retry Strategy

- Connection: 5 retries, exponential backoff (1s, 2s, 4s, 8s, 16s)
- Message send: 3 retries, 100ms delay

## Recovery Strategy

- Connection loss: Automatic reconnection
- Auth failure: Alert security, block peer
- Pipe corruption: Close and recreate

## Security Requirements

- All connections MUST authenticate with capability tokens
- Process identity MUST be verified via Windows API
- Messages from unauthenticated peers are dropped
- Capability tokens are signed and time-bounded

## Configuration

```json
{
  "ipc_protocol": {
    "pipe_prefix": "\\.\pipe\VOXY_",
    "auth_timeout_seconds": 5,
    "max_message_size_mb": 10,
    "max_connections": 50,
    "reconnect_backoff_seconds": 1,
    "enable_encryption": false
  }
}
```

## Performance Targets

| Metric | Target |
|--------|--------|
| Connection latency | < 10ms |
| Message latency (p99) | < 5ms |
| Throughput | > 10,000 msg/sec |
| Max connections | 50 |

## Resource Limits

| Resource | Limit |
|----------|-------|
| Message size | 10MB |
| Connections | 50 |
| Queue per connection | 1000 |

## Benchmarks

- `bench_connection`: Connection establishment
- `bench_latency`: Round-trip message latency
- `bench_throughput`: Max messages per second

## Testing Requirements

- Authentication tests
- Reconnection tests
- Message size limit tests
- Concurrent connection tests

## Logging Requirements

- Connections at INFO level
- Auth failures at WARN level
- Message errors at DEBUG level

## Telemetry

- Active connections
- Messages per second
- Connection duration
- Auth failure rate

## Cross References

- [012_EVENT_BUS_SPEC.md](012_EVENT_BUS_SPEC.md)
- [082_PERMISSION_MODEL_SPEC.md](082_PERMISSION_MODEL_SPEC.md)
- [090_SECURITY_MODEL_SPEC.md](090_SECURITY_MODEL_SPEC.md)

## References

- Windows Named Pipes API
- gRPC Protocol Design
- Capability-Based Security
