SECTION 11.3: Local-First Observability (OpenTelemetry)

1. Purpose

To track latency bottlenecks and errors in a production-ready way without violating user privacy by sending data to a central cloud server.

2. Responsibilities

Local-Only Metrics: Export metrics to a local SQLite database that the user can inspect via a "VOXY Health Dashboard."

Latency Tracing: Record TTFT (Time-To-First-Token) and TTA (Time-To-Audio) for every user interaction.

Error Reporting: Capture non-sensitive crash dumps (minidumps) for local debugging.

3. Architecture

We use standard OpenTelemetry but replace the OTLP exporter with a custom VoxyLocalExporter that writes to a local SQLite-backed metric store.

4. Telemetry Schema

Metric: voxy.latency.ttft, voxy.vision.ocr_accuracy, voxy.agent.route_latency.

Privacy: PII is stripped at the Rust boundary using a regex-based redaction filter before it ever hits the telemetry buffer.