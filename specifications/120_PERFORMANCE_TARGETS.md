# 120 - Performance Targets Specification

## Purpose

Define the performance Service Level Objectives (SLOs) and targets for all VOXY subsystems.

## Scope

- Latency targets
- Throughput targets
- Resource usage targets
- Availability targets
- Scalability targets

## Responsibilities

- Define measurable performance targets
- Establish SLOs
- Define SLIs (Service Level Indicators)
- Set resource budgets

## System-Wide Targets

| Metric | Target | SLO |
|--------|--------|-----|
| Cold start time | < 3s | 99th percentile |
| Memory footprint | < 2GB | Steady state |
| CPU usage (idle) | < 5% | Average |
| CPU usage (active) | < 30% | Average |
| Availability | > 99.9% | Monthly |
| Crash rate | < 0.1% | Monthly |

## Audio Pipeline Targets

| Metric | Target | Measurement |
|--------|--------|-------------|
| Wake word latency | < 200ms | End-to-end |
| STT latency (first partial) | < 500ms | 5s utterance |
| STT latency (final) | < 2s | 5s utterance |
| TTS first chunk | < 300ms | 50 tokens |
| TTS full synthesis | < 2s | 50 tokens |
| Barge-in detection | < 100ms | Voice interrupt |
| Audio pipeline throughput | Real-time | No drops |

## Cognitive Engine Targets

| Metric | Target | Measurement |
|--------|--------|-------------|
| Context assembly | < 50ms | Full assembly |
| Model routing | < 10ms | Decision time |
| First token latency | < 500ms | Cloud provider |
| First token latency (local) | < 2s | Local model |
| Token throughput | > 100/s | Generation |
| Plan generation | < 2s | Complex plan |
| Plan execution | < 5s | Per step |
| Tool call latency | < 1s | Average |

## Memory Targets

| Metric | Target | Measurement |
|--------|--------|-------------|
| Store latency | < 50ms | Single entry |
| Retrieve latency | < 100ms | Vector search |
| Consolidation | < 5min | Full cycle |
| Embedding generation | < 50ms | Single text |
| Cache hit rate | > 70% | Retrieval |

## Automation Targets

| Metric | Target | Measurement |
|--------|--------|-------------|
| UI action execution | < 2s | Complete |
| Element find | < 500ms | Average |
| Screen capture | < 200ms | Full screen |
| OCR recognition | < 200ms | Single frame |
| Object detection | < 300ms | Single frame |
| UI analysis | < 500ms | Full analysis |

## Plugin Targets

| Metric | Target | Measurement |
|--------|--------|-------------|
| Plugin load | < 2s | Complete |
| Plugin event latency | < 10ms | Routing |
| Sandbox spawn | < 1s | Complete |
| IPC latency | < 5ms | Round-trip |

## Resource Budgets

| Resource | Budget | Action on Exceed |
|----------|--------|------------------|
| RAM | 2GB | Alert, throttle |
| CPU (single core) | 50% | Throttle background |
| Disk I/O | 10MB/s | Queue |
| Network | 1MB/s | Throttle |
| Handles | 10,000 | Alert |
| Threads | 256 | Block creation |

## Scaling Targets

| Metric | Target |
|--------|--------|
| Concurrent conversations | 5 |
| Concurrent plans | 3 |
| Concurrent plugins | 50 |
| Memory entries | 1,000,000 |
| Episodic events | 100,000 |

## Cross References

- [121_BENCHMARK_SPEC.md](121_BENCHMARK_SPEC.md)
- [016_METRICS_SPEC.md](016_METRICS_SPEC.md)
- All subsystem specifications

## References

- Google SRE Book
- Performance Engineering
- Windows Performance Guidelines
