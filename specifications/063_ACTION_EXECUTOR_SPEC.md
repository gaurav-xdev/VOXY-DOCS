# 063 - Action Executor Specification

## Purpose

Define the Action Executor subsystem that performs concrete UI actions (click, type, scroll, etc.) using low-level input simulation and UIA pattern invocation.

## Scope

- Mouse input simulation
- Keyboard input simulation
- UIA pattern invocation
- Scroll actions
- Drag and drop
- Focus management
- Input validation

## Responsibilities

- Simulate mouse and keyboard input
- Invoke UIA patterns for actions
- Validate action preconditions
- Handle action errors
- Provide action feedback
- Support action recording

## Non-Goals

- Element discovery (UI Automation)
- Action planning (Planner)
- Visual analysis (Vision Runtime)

## Architecture Position

Action Executor receives resolved elements from Grounding Engine and performs the actual interactions.

## Inputs

- UIAction structs from Automation Runtime
- Resolved elements from Grounding Engine
- Action parameters

## Outputs

- Action results
- Error reports
- Action telemetry

## Interfaces

```rust
pub struct ActionExecutor {
    pub fn click(element: &UIElement, button: MouseButton) -> Result<(), ActionError>;
    pub fn type_text(element: &UIElement, text: &str) -> Result<(), ActionError>;
    pub fn key_press(key: Key) -> Result<(), ActionError>;
    pub fn scroll(element: &UIElement, direction: ScrollDirection, amount: i32) -> Result<(), ActionError>;
    pub fn drag_and_drop(source: &UIElement, target: &UIElement) -> Result<(), ActionError>;
    pub fn set_focus(element: &UIElement) -> Result<(), ActionError>;
}
```

## Data Models

### ActionResult

| Field | Type | Description |
|-------|------|-------------|
| action_id | UUID | Action identifier |
| success | bool | Action succeeded |
| error | Option<ActionError> | Error if failed |
| duration_ms | u64 | Execution time |
| before_state | Option<UIState> | State before |
| after_state | Option<UIState> | State after |

## State Management

- No persistent state
- Action history maintained in Working Memory

## Threading Model

Actions run on the UIA STA thread or dedicated input thread.

## IPC Requirements

- No IPC for actions

## Event Definitions

| Event | Payload | Description |
|-------|---------|-------------|
| `action.executed` | ActionResult | Action completed |
| `action.failed` | (action, error) | Action failed |

## Error Handling

| Code | Description | Recovery |
|------|-------------|----------|
| `AE001` | Element not interactable | Scroll into view, retry |
| `AE002` | Input blocked | Wait, retry |
| `AE003` | Coordinate out of bounds | Adjust, retry |
| `AE004` | SendInput failed | Retry once |

## Retry Strategy

- Action: 2 retries
- Input: 1 retry

## Recovery Strategy

- Not interactable: Scroll, retry
- Blocked: Wait, retry
- Out of bounds: Recalculate

## Security Requirements

- Actions require user consent for sensitive operations
- No input to UAC or security dialogs
- Input rate limited to prevent abuse

## Configuration

```json
{
  "action_executor": {
    "action_timeout_ms": 5000,
    "input_delay_ms": 10,
    "retry_count": 2,
    "scroll_into_view": true,
    "verify_after_action": true
  }
}
```

## Performance Targets

| Metric | Target |
|--------|--------|
| Click latency | < 100ms |
| Type latency | < 50ms per char |
| Scroll latency | < 200ms |

## Resource Limits

| Resource | Limit |
|----------|-------|
| Concurrent actions | 1 |
| Input rate | 100/sec |

## Benchmarks

- `bench_click`: Click speed
- `bench_type`: Typing speed
- `bench_scroll`: Scroll speed

## Testing Requirements

- Action accuracy tests
- Retry tests
- Error handling tests

## Logging Requirements

- Actions at DEBUG level
- Errors at ERROR level

## Telemetry

- Actions executed
- Average latency
- Error rate
- Retry count

## Cross References

- [060_AUTOMATION_RUNTIME_SPEC.md](060_AUTOMATION_RUNTIME_SPEC.md)
- [061_GROUNDING_ENGINE_SPEC.md](061_GROUNDING_ENGINE_SPEC.md)
- [062_UI_AUTOMATION_SPEC.md](062_UI_AUTOMATION_SPEC.md)

## References

- Windows SendInput API
- UIA Pattern Invocation
- Input Simulation Best Practices
