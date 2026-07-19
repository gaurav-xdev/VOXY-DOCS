# 121 - Benchmark Specification

## Purpose

Define the benchmark methodology, test suites, and measurement protocols for evaluating VOXY performance against established targets.

## Scope

- Benchmark definitions
- Test data sets
- Measurement protocols
- Reporting formats
- Regression detection
- CI integration

## Responsibilities

- Define benchmark suites
- Provide test data
- Standardize measurements
- Detect regressions
- Generate reports
- Integrate with CI

## Benchmark Suites

### Kernel Benchmarks

| Benchmark | Description | Target |
|-----------|-------------|--------|
| `bench_kernel_startup` | Cold start to Ready | < 3s |
| `bench_subsystem_init` | Per-subsystem init | < 2s |
| `bench_thread_pool` | Task scheduling p99 | < 10ms |

### Audio Benchmarks

| Benchmark | Description | Target |
|-----------|-------------|--------|
| `bench_capture_latency` | Frame delivery | < 20ms |
| `bench_wake_word` | Detection latency | < 200ms |
| `bench_vad_accuracy` | Precision/recall | > 95% |
| `bench_stt_latency` | End-to-end | < 2s |
| `bench_tts_first_chunk` | First audio | < 300ms |

### Cognitive Benchmarks

| Benchmark | Description | Target |
|-----------|-------------|--------|
| `bench_context_assembly` | Assembly time | < 50ms |
| `bench_model_routing` | Decision time | < 10ms |
| `bench_inference_latency` | First token | < 500ms |
| `bench_plan_generation` | Plan creation | < 2s |
| `bench_tool_dispatch` | Tool call | < 1s |

### Memory Benchmarks

| Benchmark | Description | Target |
|-----------|-------------|--------|
| `bench_store` | Storage latency | < 50ms |
| `bench_retrieve` | Search latency | < 100ms |
| `bench_embedding` | Generation | < 50ms |
| `bench_consolidation` | Full cycle | < 5min |

### Automation Benchmarks

| Benchmark | Description | Target |
|-----------|-------------|--------|
| `bench_action` | Execution time | < 2s |
| `bench_find` | Element find | < 500ms |
| `bench_capture` | Screen capture | < 200ms |
| `bench_ocr` | Recognition | < 200ms |

## Measurement Protocol

1. **Warm-up**: 5 iterations before measurement
2. **Iterations**: Minimum 100 iterations
3. **Outliers**: Remove top/bottom 1%
4. **Reporting**: P50, P95, P99, mean, stddev
5. **Environment**: Document hardware, OS, load

## Test Data

- Audio: LibriSpeech samples
- Text: Standard NLP datasets
- UI: Synthetic Windows applications
- Images: Standard vision datasets

## Regression Detection

- Baseline stored per release
- 10% regression triggers alert
- 20% regression blocks release
- Automated CI comparison

## Reporting Format

```json
{
  "benchmark": "bench_stt_latency",
  "timestamp": "2026-07-17T21:00:00Z",
  "environment": {
    "cpu": "Intel i7-12700K",
    "ram": "32GB",
    "os": "Windows 11 23H2"
  },
  "results": {
    "iterations": 100,
    "mean_ms": 450,
    "p50_ms": 420,
    "p95_ms": 580,
    "p99_ms": 650,
    "stddev_ms": 80
  },
  "target_ms": 500,
  "status": "pass"
}
```

## Cross References

- [120_PERFORMANCE_TARGETS.md](120_PERFORMANCE_TARGETS.md)
- [130_TESTING_SPEC.md](130_TESTING_SPEC.md)

## References

- Google Benchmark
- Criterion.rs
- Performance Testing Best Practices
