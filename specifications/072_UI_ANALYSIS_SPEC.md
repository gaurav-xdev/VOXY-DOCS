# 072 - UI Analysis Specification

## Purpose

Define the UI Analysis subsystem that understands the structure, layout, and functionality of user interfaces by combining vision, OCR, and UI automation data.

## Scope

- UI structure extraction
- Layout analysis
- Functionality inference
- Accessibility tree enrichment
- UI state description generation
- Change detection

## Responsibilities

- Analyze UI structure
- Describe UI state in natural language
- Detect UI changes
- Enrich accessibility data
- Generate UI summaries
- Support grounding queries

## Non-Goals

- UI automation execution
- Screen capture
- General vision tasks

## Architecture Position

UI Analysis combines data from UI Automation, Object Detection, and OCR to provide comprehensive UI understanding.

## Inputs

- UI automation trees
- Object detection results
- OCR text
- Screen captures

## Outputs

- UI structure descriptions
- UI state summaries
- Change reports
- Element functionality descriptions

## Interfaces

```rust
pub struct UIAnalysis {
    pub fn analyze(ui_tree: &UITree, vision: &VisionResult) -> Result<UIState, AnalysisError>;
    pub fn describe_state(state: &UIState) -> String;
    pub fn detect_changes(old: &UIState, new: &UIState) -> Vec<UIChange>;
    pub fn find_element_by_function(function: &str, state: &UIState) -> Option<UIElement>;
}
```

## Data Models

### UIState

| Field | Type | Description |
|-------|------|-------------|
| window | WindowInfo | Active window |
| elements | Vec<AnalyzedElement> | Analyzed elements |
| layout | Layout | Layout description |
| summary | String | NL summary |

### AnalyzedElement

| Field | Type | Description |
|-------|------|-------------|
| element | UIElement | Base element |
| detected_class | Option<String> | Vision class |
| ocr_text | Option<String> | OCR text |
| functionality | Vec<String> | Inferred functions |
| importance | f32 | Element importance |

### UIChange

| Field | Type | Description |
|-------|------|-------------|
| type | ChangeType | Added, Removed, Modified |
| element | AnalyzedElement | Changed element |
| description | String | Change description |

## State Management

- Previous UI state cached for change detection
- No persistent state

## Threading Model

Analysis runs on blocking pool.

## IPC Requirements

- No IPC

## Event Definitions

| Event | Payload | Description |
|-------|---------|-------------|
| `ui.analysis_complete` | UIState | Analysis done |
| `ui.changes_detected` | Vec<UIChange> | Changes found |

## Error Handling

| Code | Description | Recovery |
|------|-------------|----------|
| `UA001` | Tree parse error | Use vision only |
| `UA002` | Analysis timeout | Return partial |

## Retry Strategy

- Analysis: 1 retry

## Recovery Strategy

- Parse error: Vision fallback
- Timeout: Partial results

## Security Requirements

- No special requirements

## Configuration

```json
{
  "ui_analysis": {
    "enable_vision_enrichment": true,
    "enable_ocr_enrichment": true,
    "change_detection_enabled": true,
    "analysis_timeout_ms": 2000,
    "summary_max_length": 500
  }
}
```

## Performance Targets

| Metric | Target |
|--------|--------|
| Analysis latency | < 500ms |
| Change detection | < 100ms |
| Summary generation | < 200ms |

## Resource Limits

| Resource | Limit |
|----------|-------|
| Element cache | 1000 |
| Summary length | 500 chars |

## Benchmarks

- `bench_analysis`: Analysis speed
- `bench_changes`: Change detection

## Testing Requirements

- Structure accuracy tests
- Change detection tests
- Summary quality tests

## Logging Requirements

- Analysis at DEBUG level
- Changes at INFO level

## Telemetry

- Analysis frequency
- Average latency
- Change frequency

## Cross References

- [070_VISION_RUNTIME_SPEC.md](070_VISION_RUNTIME_SPEC.md)
- [062_UI_AUTOMATION_SPEC.md](062_UI_AUTOMATION_SPEC.md)
- [061_GROUNDING_ENGINE_SPEC.md](061_GROUNDING_ENGINE_SPEC.md)

## References

- UI Understanding Research
- Accessibility Tree Analysis
- Layout Analysis
