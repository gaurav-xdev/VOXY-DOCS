# 065 - OCR Specification

## Purpose

Define the OCR subsystem that extracts text from screen captures and images using both Windows-built-in OCR and local ONNX models.

## Scope

- Text extraction from images
- Windows OCR API integration
- Local ONNX OCR model support
- Multi-language OCR
- Text region detection
- Confidence scoring

## Responsibilities

- Extract text from screen captures
- Support multiple languages
- Provide text bounding boxes
- Score recognition confidence
- Handle low-quality images
- Cache OCR results

## Non-Goals

- Document OCR (future enhancement)
- Handwriting recognition
- Layout analysis

## Architecture Position

OCR processes frames from Screen Capture and provides text to Grounding Engine and UI Analysis.

## Inputs

- Images from Screen Capture
- Language hints
- Region of interest

## Outputs

- Extracted text with bounding boxes
- Confidence scores
- Language detection

## Interfaces

```rust
pub struct OCR {
    pub fn recognize(image: &Image, lang: &str) -> Result<OCRResult, OCRError>;
    pub fn recognize_regions(image: &Image, regions: &[Region]) -> Result<Vec<OCRResult>, OCRError>;
    pub fn detect_text_regions(image: &Image) -> Result<Vec<Region>, OCRError>;
    pub fn get_supported_languages() -> Vec<String>;
}
```

## Data Models

### OCRResult

| Field | Type | Description |
|-------|------|-------------|
| text | String | Recognized text |
| confidence | f32 | Recognition confidence |
| bounding_box | Rect | Text location |
| language | String | Detected language |
| lines | Vec<TextLine> | Per-line results |

### TextLine

| Field | Type | Description |
|-------|------|-------------|
| text | String | Line text |
| bounding_box | Rect | Line location |
| words | Vec<WordInfo> | Per-word info |

## State Management

- OCR models loaded on demand
- Results cached with TTL
- No persistent state

## Threading Model

OCR runs on blocking pool. Models are loaded once and reused.

## IPC Requirements

- No IPC for OCR

## Event Definitions

| Event | Payload | Description |
|-------|---------|-------------|
| `ocr.completed` | OCRResult | OCR complete |
| `ocr.failed` | OCRError | OCR failed |

## Error Handling

| Code | Description | Recovery |
|------|-------------|----------|
| `OC001` | Model not loaded | Load model, retry |
| `OC002` | Language not supported | Use default language |
| `OC003` | Image too large | Resize, retry |

## Retry Strategy

- Recognition: 1 retry
- Model load: 2 retries

## Recovery Strategy

- Model failure: Use Windows OCR
- Language failure: Use English

## Security Requirements

- OCR data not logged
- No external transmission

## Configuration

```json
{
  "ocr": {
    "primary_engine": "windows",
    "fallback_engine": "onnx",
    "default_language": "en",
    "confidence_threshold": 0.6,
    "cache_enabled": true,
    "cache_ttl_seconds": 60
  }
}
```

## Performance Targets

| Metric | Target |
|--------|--------|
| Recognition latency | < 200ms |
| Accuracy | > 95% |
| Memory per image | < 50MB |

## Resource Limits

| Resource | Limit |
|----------|-------|
| Model memory | 200MB |
| Image size | 4K |
| Cache size | 100 |

## Benchmarks

- `bench_recognition`: Recognition speed
- `bench_accuracy`: Accuracy on test set

## Testing Requirements

- Accuracy tests
- Language tests
- Error handling tests

## Logging Requirements

- Recognition at TRACE level
- Errors at ERROR level

## Telemetry

- Recognition frequency
- Average latency
- Accuracy

## Cross References

- [060_AUTOMATION_RUNTIME_SPEC.md](060_AUTOMATION_RUNTIME_SPEC.md)
- [064_SCREEN_CAPTURE_SPEC.md](064_SCREEN_CAPTURE_SPEC.md)
- [072_UI_ANALYSIS_SPEC.md](072_UI_ANALYSIS_SPEC.md)

## References

- Windows OCR API
- ONNX Runtime OCR
- Tesseract OCR
