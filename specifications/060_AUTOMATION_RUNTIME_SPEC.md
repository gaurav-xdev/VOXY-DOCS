# 060 - Automation Runtime Specification

## Purpose

Define the Automation Runtime subsystem that orchestrates UI automation, screen capture, OCR, and action execution to enable VOXY to interact with the Windows desktop and applications.

## Scope

- UI automation orchestration
- Screen capture coordination
- OCR pipeline management
- Action execution scheduling
- Element resolution and caching
- Automation state management

## Responsibilities

- Coordinate automation subsystems
- Manage automation state
- Schedule and execute actions
- Handle automation errors
- Provide automation diagnostics
- Enforce automation safety limits

## Non-Goals

- Specific UI automation logic (UI Automation subsystem)
- Screen capture implementation (Screen Capture subsystem)
- OCR algorithms (OCR subsystem)
- Action implementation (Action Executor)

## Architecture Position

Automation Runtime sits above Grounding Engine, UI Automation, Screen Capture, OCR, and Action Executor. It is the coordinator for all desktop automation.

## Inputs

- Action plans from Planner
- UI element queries
- Screen capture requests
- User override commands

## Outputs

- Action execution results
- UI state descriptions
- Screen content analysis
- Automation status events

## Interfaces

```rust
pub struct AutomationRuntime {
    pub fn execute_action(action: UIAction) -> Result<ActionResult, AutomationError>;
    pub fn get_ui_state() -> Result<UIState, AutomationError>;
    pub fn capture_screen(region: Option<Region>) -> Result<ScreenCapture, AutomationError>;
    pub fn find_element(query: ElementQuery) -> Result<Option<UIElement>, AutomationError>;
    pub fn wait_for_condition(condition: Condition, timeout: Duration) -> Result<bool, AutomationError>;
}
```

## Data Models

### UIAction

| Field | Type | Description |
|-------|------|-------------|
| id | UUID | Action identifier |
| type | ActionType | Click, Type, Scroll, etc. |
| target | Option<ElementRef> | Target element |
| parameters | Map | Action parameters |
| timeout_ms | u64 | Action timeout |

### UIState

| Field | Type | Description |
|-------|------|-------------|
| foreground_window | WindowInfo | Active window |
| elements | Vec<UIElement> | Visible elements |
| focused_element | Option<ElementRef> | Focused element |
| timestamp | DateTime | Capture time |

### ActionType

| Variant | Description |
|---------|-------------|
| `Click` | Mouse click |
| `DoubleClick` | Double click |
| `RightClick` | Right mouse click |
| `Type` | Keyboard input |
| `KeyPress` | Single key press |
| `Scroll` | Scroll action |
| `Focus` | Set focus |
| `Select` | Select item |
| `Drag` | Drag and drop |
| `Wait` | Wait for condition |

## State Management

- Automation state tracks current operation
- Element cache maintained for performance
- Screen state cached with TTL

## Threading Model

Automation runs on dedicated threads (UI automation requires STA). Coordination is async via channels.

## IPC Requirements

- No IPC for automation
- External tools may request automation via IPC

## Event Definitions

| Event | Payload | Description |
|-------|---------|-------------|
| `automation.action.started` | UIAction | Action started |
| `automation.action.completed` | ActionResult | Action completed |
| `automation.action.failed` | (action, error) | Action failed |
| `automation.state.changed` | UIState | UI state changed |
| `automation.screen.captured` | ScreenCapture | Screen captured |

## Error Handling

| Code | Description | Recovery |
|------|-------------|----------|
| `AR001` | Element not found | Retry with alternative selector |
| `AR002` | Action timeout | Cancel, return timeout |
| `AR003` | Window closed | Refresh state, retry |
| `AR004` | Permission denied | Skip action, notify user |
| `AR005` | Automation API error | Retry once, then fail |

## Retry Strategy

- Element find: 3 retries with 500ms delay
- Action execution: 2 retries
- Screen capture: 2 retries

## Recovery Strategy

- Element not found: Try alternative selectors, then fail
- Window closed: Refresh UI state, retry
- API error: Retry once, then skip

## Security Requirements

- Automation requires user consent
- Sensitive actions require confirmation
- No automation of security dialogs
- UAC prompts cannot be automated

## Configuration

```json
{
  "automation_runtime": {
    "action_timeout_seconds": 10,
    "element_find_retries": 3,
    "screen_capture_interval_ms": 500,
    "element_cache_ttl_ms": 1000,
    "max_concurrent_actions": 1,
    "require_confirmation_for": ["file_delete", "install", "run_as_admin"]
  }
}
```

## Performance Targets

| Metric | Target |
|--------|--------|
| Action execution | < 2s |
| Element find | < 500ms |
| Screen capture | < 200ms |
| UI state refresh | < 1s |

## Resource Limits

| Resource | Limit |
|----------|-------|
| Concurrent actions | 1 |
| Element cache | 1000 |
| Screen cache | 10 |

## Benchmarks

- `bench_action`: Action execution time
- `bench_find`: Element find time
- `bench_capture`: Screen capture time

## Testing Requirements

- Action execution tests
- Element resolution tests
- Screen capture tests
- Error recovery tests

## Logging Requirements

- Actions at INFO level
- Errors at ERROR level
- State changes at DEBUG level

## Telemetry

- Actions executed
- Average execution time
- Error rate
- Screen captures

## Cross References

- [061_GROUNDING_ENGINE_SPEC.md](061_GROUNDING_ENGINE_SPEC.md)
- [062_UI_AUTOMATION_SPEC.md](062_UI_AUTOMATION_SPEC.md)
- [063_ACTION_EXECUTOR_SPEC.md](063_ACTION_EXECUTOR_SPEC.md)
- [064_SCREEN_CAPTURE_SPEC.md](064_SCREEN_CAPTURE_SPEC.md)
- [065_OCR_SPEC.md](065_OCR_SPEC.md)

## References

- Microsoft UI Automation
- Windows Graphics Capture
- Accessibility Testing Patterns
