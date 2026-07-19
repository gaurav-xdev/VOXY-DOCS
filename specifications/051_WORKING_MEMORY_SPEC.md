# 051 - Working Memory Specification

## Purpose

Define the Working Memory subsystem that maintains short-term, session-specific context including conversation history, active tasks, and temporary state.

## Scope

- Conversation history management
- Session state tracking
- Active task context
- Temporary variable storage
- Context window management
- Session boundaries

## Responsibilities

- Store conversation turns
- Track active tasks and plans
- Maintain session variables
- Manage context window size
- Handle session start/end
- Provide quick access to recent context

## Non-Goals

- Long-term persistence (Episodic/Semantic Memory)
- Knowledge storage (Semantic Memory)
- Skill storage (Procedural Memory)

## Architecture Position

Working Memory is the fastest, most accessible memory tier. It feeds directly into Context Assembly.

## Inputs

- Conversation turns
- Task state updates
- Session events
- Context requests

## Outputs

- Conversation history
- Session state
- Active task context
- Working memory snapshots

## Interfaces

```rust
pub struct WorkingMemory {
    pub fn add_turn(turn: ConversationTurn);
    pub fn get_history(limit: usize) -> Vec<ConversationTurn>;
    pub fn set_variable(key: &str, value: &str);
    pub fn get_variable(key: &str) -> Option<String>;
    pub fn clear();
    pub fn snapshot() -> WorkingMemorySnapshot;
}
```

## Data Models

### ConversationTurn

| Field | Type | Description |
|-------|------|-------------|
| id | UUID | Turn identifier |
| role | Role | User, Assistant, System |
| content | String | Message content |
| timestamp | DateTime | Turn time |
| metadata | Map | Additional data |

### WorkingMemorySnapshot

| Field | Type | Description |
|-------|------|-------------|
| turns | Vec<ConversationTurn> | Recent conversation |
| variables | Map | Session variables |
| active_tasks | Vec<Task> | Active tasks |
| token_count | usize | Approximate tokens |

## State Management

- Working memory is in-memory only
- Cleared on session end
- Limited by token budget

## Threading Model

Working memory is accessed via `RwLock` for thread safety.

## IPC Requirements

- No IPC for working memory

## Event Definitions

| Event | Payload | Description |
|-------|---------|-------------|
| `wm.turn_added` | ConversationTurn | Turn added |
| `wm.variable_set` | (key, value) | Variable set |
| `wm.cleared` | {} | Memory cleared |
| `wm.session_ended` | session_id | Session ended |

## Error Handling

| Code | Description | Recovery |
|------|-------------|----------|
| `WM001` | Token budget exceeded | Truncate oldest turns |
| `WM002` | Variable too large | Reject, alert |

## Retry Strategy

- None (in-memory)

## Recovery Strategy

- Budget exceeded: Truncate history
- Overflow: Clear and restart

## Security Requirements

- Working memory is process-local
- No persistence without explicit action

## Configuration

```json
{
  "working_memory": {
    "max_turns": 100,
    "max_token_budget": 4000,
    "variable_max_size_kb": 10,
    "session_timeout_minutes": 30
  }
}
```

## Performance Targets

| Metric | Target |
|--------|--------|
| Add turn latency | < 1ms |
| History retrieval | < 5ms |
| Token count | < 10ms |

## Resource Limits

| Resource | Limit |
|----------|-------|
| Max turns | 100 |
| Token budget | 4000 |
| Variables | 100 |

## Benchmarks

- `bench_add_turn`: Turn insertion
- `bench_retrieval`: History access

## Testing Requirements

- Token budget tests
- Truncation tests
- Session boundary tests

## Logging Requirements

- Session events at INFO level
- Turns at DEBUG level

## Telemetry

- Turn count
- Token usage
- Session duration

## Cross References

- [050_MEMORY_RUNTIME_SPEC.md](050_MEMORY_RUNTIME_SPEC.md)
- [040_CONTEXT_ASSEMBLY_SPEC.md](040_CONTEXT_ASSEMBLY_SPEC.md)

## References

- Working Memory (Cognitive Psychology)
- Conversation State Management
