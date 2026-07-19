# 062 - UI Automation Specification

## Purpose

Define the UI Automation subsystem that integrates with Microsoft UI Automation (UIA) to inspect, navigate, and interact with Windows applications and UI elements.

## Scope

- UIA client implementation
- UI tree traversal and inspection
- Element property access
- Pattern invocation (Invoke, Value, Selection, etc.)
- Event subscription (structure changes, focus changes)
- Element caching and refresh

## Responsibilities

- Discover UI elements via UIA
- Read element properties
- Invoke UIA patterns
- Subscribe to UI events
- Maintain element cache
- Handle application lifecycle

## Non-Goals

- Direct input simulation (Action Executor)
- Screen capture (Screen Capture)
- Visual analysis (Vision Runtime)

## Architecture Position

UI Automation is the primary interface to Windows applications. It provides structured access to UI elements.

## Inputs

- Element queries from Grounding Engine
- Pattern invocation requests from Action Executor
- Event subscriptions from Automation Runtime

## Outputs

- UI element trees
- Element properties
- Event notifications
- Pattern invocation results

## Interfaces

```rust
pub struct UIAutomation {
    pub fn get_root_element() -> UIElement;
    pub fn find_element(query: &ElementQuery) -> Result<Option<UIElement>, UIAError>;
    pub fn find_elements(query: &ElementQuery) -> Result<Vec<UIElement>, UIAError>;
    pub fn get_element_properties(element: &UIElement) -> ElementProperties;
    pub fn invoke_pattern(element: &UIElement, pattern: Pattern) -> Result<(), UIAError>;
    pub fn subscribe_event(event: UIEventType, handler: EventHandler) -> Subscription;
}
```

## Data Models

### UIElement

| Field | Type | Description |
|-------|------|-------------|
| automation_id | String | UIA automation ID |
| name | String | Element name |
| control_type | ControlType | Button, Edit, etc. |
| class_name | String | Window class |
| bounding_rect | Rect | Screen coordinates |
| enabled | bool | Is enabled |
| visible | bool | Is visible |
| patterns | Vec<Pattern> | Supported patterns |

### ElementQuery

| Field | Type | Description |
|-------|------|-------------|
| automation_id | Option<String> | Match automation ID |
| name | Option<String> | Match name |
| control_type | Option<ControlType> | Match control type |
| class_name | Option<String> | Match class |
| scope | SearchScope | Children, Descendants, etc. |

### Pattern

| Variant | Description |
|---------|-------------|
| `Invoke` | Click/button press |
| `Value` | Get/set value |
| `Selection` | Select item |
| `ExpandCollapse` | Expand/collapse |
| `Scroll` | Scroll |
| `Window` | Window operations |

## State Management

- Element cache maintained with TTL
- Event subscriptions tracked
- No persistent state

## Threading Model

UIA requires STA (Single Threaded Apartment). All UIA operations run on a dedicated STA thread.

## IPC Requirements

- No IPC for UIA

## Event Definitions

| Event | Payload | Description |
|-------|---------|-------------|
| `uia.element_found` | UIElement | Element discovered |
| `uia.structure_changed` | element | UI structure changed |
| `uia.focus_changed` | element | Focus changed |
| `uia.property_changed` | (element, property) | Property changed |

## Error Handling

| Code | Description | Recovery |
|------|-------------|----------|
| `UA001` | Element not found | Return None |
| `UA002` | Pattern not supported | Return error |
| `UA003` | UIA COM error | Retry once |
| `UA004` | Element stale | Refresh cache, retry |

## Retry Strategy

- Element find: 2 retries
- Pattern invoke: 1 retry
- COM error: 1 retry

## Recovery Strategy

- Stale element: Refresh and retry
- COM error: Retry, then fail

## Security Requirements

- UIA respects Windows security boundaries
- Elevated elements not accessible without elevation
- UAC dialogs cannot be automated

## Configuration

```json
{
  "ui_automation": {
    "cache_ttl_ms": 1000,
    "element_find_timeout_ms": 5000,
    "pattern_invoke_timeout_ms": 5000,
    "event_buffer_size": 100,
    "max_tree_depth": 50
  }
}
```

## Performance Targets

| Metric | Target |
|--------|--------|
| Element find | < 500ms |
| Property read | < 50ms |
| Pattern invoke | < 100ms |
| Tree traversal | < 1s |

## Resource Limits

| Resource | Limit |
|----------|-------|
| Cache size | 1000 |
| Tree depth | 50 |
| Event subscriptions | 50 |

## Benchmarks

- `bench_find`: Element find speed
- `bench_traverse`: Tree traversal
- `bench_invoke`: Pattern invocation

## Testing Requirements

- Element discovery tests
- Pattern invocation tests
- Event handling tests
- Cache tests

## Logging Requirements

- Finds at TRACE level
- Invocations at DEBUG level
- Errors at ERROR level

## Telemetry

- Elements found
- Patterns invoked
- Event frequency
- Cache hit rate

## Cross References

- [060_AUTOMATION_RUNTIME_SPEC.md](060_AUTOMATION_RUNTIME_SPEC.md)
- [061_GROUNDING_ENGINE_SPEC.md](061_GROUNDING_ENGINE_SPEC.md)
- [063_ACTION_EXECUTOR_SPEC.md](063_ACTION_EXECUTOR_SPEC.md)

## References

- Microsoft UI Automation
- IUIAutomation Interface
- UIA Control Patterns
