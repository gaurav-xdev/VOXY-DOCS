VOXY: Comprehensive Architectural Overview

This document serves as the master map for the VOXY project, synthesizing our research into legacy systems, our three-phase implementation strategy, and the modular packages that define our engineering roadmap.

1. Legacy Analysis & Foundational Lessons

VOXY is designed by auditing the successes and failures of three key predecessor architectures. We inherit the strengths of these systems while strictly mitigating their weaknesses.

Siri (The Latency Benchmark):

Strength: Perceived ultra-low latency via custom silicon (AOP).

Weakness: Cloud-tethered reliance and lack of context window/agentic flexibility.

VOXY Adaptation: We achieve "Siri-beating" latency ($<500\text{ms}$) via raw Windows WASAPI exclusive-mode integration, local inference, and predictive audio chunking.

OpenClaw (The Agentic Control Benchmark):

Strength: Demonstrates the power of unconstrained agentic desktop control.

Weakness: Significant security risks (direct shell execution/prompt injection) and brittle, hardcoded dependencies.

VOXY Adaptation: We replace raw script generation with an MCP (Model Context Protocol) toolset, running high-risk actions inside an isolated Windows Sandbox (WSB).

Hermes (The Continuous Learning Benchmark):

Strength: Continuous procedural memory and self-improvement loops.

Weakness: "Vector Soup" memory bloating and hallucinated procedural scripts.

VOXY Adaptation: We utilize a hybrid relational-vector memory engine (sqlite-vec) for deterministic accuracy and compile workflows into safe, validated JSON State Machines.

2. Implementation Strategy: The Phases

Our engineering follows a logical progression from low-level kernel stability to high-level cognitive memory.

Phase 1: Foundation (The Kernel)

Focus: Rust-based daemon, event-driven architecture, and zero-copy audio pipelines.

Key Deliverable: The stable Rust Kernel that survives Windows power transitions.

Phase 2: Intelligence & Vision (The Brain & Eyes)

Focus: Multi-agent routing, DPI-aware screen interaction, and local OCR.

Key Deliverable: The ability to "see" and "think" across mixed-monitor setups.

Phase 3: Presentation & Lifecycle (The Body)

Focus: WinUI 3 interface, telemetry, and power lifecycle hooks.

Key Deliverable: A polished, resilient desktop presence.

3. The Three Engineering Packages

To reach production, we have organized the remaining backlog into three distinct engineering packages.

Package A: The Multi-Agent AI Engine (Cognitive)

Goal: Define how VOXY "thinks" and routes intents.

Components: The Intent Router (Phi-3 Gatekeeper), Token Budgeting (Context Assembler), and Model Fallback Strategy (DirectML vs Cloud).

Package B: Advanced Vision & Understanding (Sensory)

Goal: Empower VOXY to see the screen without lagging the system.

Components: Zero-copy OCR capture via Windows.Graphics.Capture, DPI-safe coordinate mapping, and VLM Fallback (Moondream2) for when UIA/OCR fails.

Package C: Presentation & Observability (Lifecycle)

Goal: Create a visual interface that is performant and reliable.

Components: The gRPC WinUI 3 overlay, Windows Power Lifecycle hooks (PBT_APMSUSPEND), and Local-First OpenTelemetry metrics.

4. Security & Privacy Mandate

Regardless of the phase or package, all VOXY development adheres to the "Local-First" Security Protocol:

Sandboxing: High-risk actions must use WSB.

Deterministic Inputs: Avoid raw shell scripts; use JSON-based MCP tools.

Data Residency: All logs, memories, and telemetry are stored in a local SQLite file.