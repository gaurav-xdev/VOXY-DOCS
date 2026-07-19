# 061 - Grounding Engine Specification

## Purpose

Define the Grounding Engine subsystem that resolves natural language references to specific UI elements, screen regions, and application objects using vision, OCR, and UI automation data.

## Scope

- Natural language to UI element resolution
- Screen region grounding
- Application object identification
- Multi-modal grounding (text + vision)
- Grounding confidence scoring
- Fallback strategies

## Responsibilities

- Resolve NL references to UI elements
- Ground spatial references ("the button on the right")
- Combine UI automation and vision data
- Score grounding confidence
- Provide fallback resolutions
- Cache grounding results

## Non-Goals

- UI automation execution (Action Executor)
- Screen capture (Screen Capture)
- OCR processing (OCR subsystem)

## Architecture Position

Grounding Engine sits between Planner/Executor and UI Automation/Vision. It translates abstract references into concrete UI targets.

## Inputs

- Natural language references
- UI automation trees
- Screen captures
- OCR text results
- Historical grounding data

## Outputs

- Resolved UI elements
- Grounding confidence scores
- Alternative candidates
- Grounding explanations

## Interfaces

```rust
pub struct GroundingEngine {
    pub fn ground_element(reference: &str, context: &GroundingContext) -> Result<GroundedElement, GroundingError>;
    pub fn ground_region(description: &str, screen: &ScreenCapture) -> Result<Region, GroundingError>;
    pub fn get_candidates(reference: &str) -> Vec<GroundedElement>;
    pub fn explain_grounding(result: &GroundedElement) -> String;
}
```

## Data Models

### GroundedElement

| Field | Type | Description |
|-------|------|-------------|
| element | UIElement | Resolved element |
| confidence | f32 | Grounding confidence |
| method | GroundingMethod | How it was resolved |
| alternatives | Vec<UIElement> | Other candidates |
| explanation | String | Human-readable explanation |

### GroundingMethod

| Variant | Description |
|---------|-------------|
| `ExactMatch` | Exact text match |
| `PartialMatch` | Partial text match |
| `SemanticMatch` | Semantic similarity |
| `Spatial` | Spatial relationship |
| `Visual` | Visual recognition |
| `Historical` | Previously used element |

## State Management

- Grounding cache maintained
- Historical groundings stored
- No persistent state beyond cache

## Threading Model

Grounding runs on blocking pool. May use async for model inference.

## IPC Requirements

- No IPC for grounding

## Event Definitions

| Event | Payload | Description |
|-------|---------|-------------|
| `grounding.resolved` | GroundedElement | Element grounded |
| `grounding.failed` | reference | Grounding failed |
| `grounding.cache_hit` | reference | Cache hit |

## Error Handling

| Code | Description | Recovery |
|------|-------------|----------|
| `GE001` | No match found | Return alternatives, ask user |
| `GE002` | Ambiguous match | Return candidates, ask user |
| `GE003` | Confidence too low | Return with warning |

## Retry Strategy

- Grounding: 1 retry with relaxed matching

## Recovery Strategy

- No match: Try fuzzy matching, then ask user
- Ambiguous: Present options
- Low confidence: Warn and proceed

## Security Requirements

- Grounding respects UI automation security
- No access to elevated elements

## Configuration

```json
{
  "grounding_engine": {
    "min_confidence": 0.7,
    "max_candidates": 5,
    "cache_enabled": true,
    "cache_ttl_seconds": 300,
    "enable_semantic_matching": true,
    "enable_visual_grounding": true,
    "fuzzy_match_threshold": 0.6
  }
}
```

## Performance Targets

| Metric | Target |
|--------|--------|
| Grounding latency | < 500ms |
| Cache hit rate | > 60% |
| Confidence accuracy | > 85% |

## Resource Limits

| Resource | Limit |
|----------|-------|
| Cache size | 500 |
| Candidates | 5 |

## Benchmarks

- `bench_grounding`: Resolution speed
- `bench_accuracy`: Grounding accuracy

## Testing Requirements

- Resolution accuracy tests
- Ambiguity handling tests
- Cache tests

## Logging Requirements

- Resolutions at DEBUG level
- Failures at WARN level

## Telemetry

- Grounding frequency
- Average confidence
- Cache hit rate
- Failure rate

## Cross References

- [060_AUTOMATION_RUNTIME_SPEC.md](060_AUTOMATION_RUNTIME_SPEC.md)
- [062_UI_AUTOMATION_SPEC.md](062_UI_AUTOMATION_SPEC.md)
- [072_UI_ANALYSIS_SPEC.md](072_UI_ANALYSIS_SPEC.md)

## References

- Natural Language Grounding
- UI Element Resolution
- Multi-Modal AI
