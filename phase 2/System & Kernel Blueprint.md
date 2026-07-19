VOXY Master Blueprint: System Architecture & Kernel

Covers: System Architecture, VOXY Kernel, Windows Integration, Performance

1. Purpose

To define the central runtime environment, process model, and system-level lifecycle of VOXY, ensuring an event-driven, secure, and ultra-low-latency foundation natively integrated with Windows.

2. Responsibilities

Process Lifecycle: Manage startup, shutdown, and recovery of all subsystems.

Concurrency & Scheduling: Allocate thread priorities (e.g., Audio thread at THREAD_PRIORITY_TIME_CRITICAL).

Event Routing: Broker asynchronous messages between disjointed domains (Audio, AI, UI, OS).

Windows Interop: Manage the background Windows Service, System Tray lifecycle, and OS power state hooks.

3. Architecture

Based on Phase 1 ADR 001, VOXY Core is implemented in Rust using tokio for async scheduling and windows-rs for native API access.

graph TD
    subgraph UI Process
        A[WinUI 3 Tray/Overlay]
    end

    subgraph VOXY Kernel Process [Rust Daemon - Standard User]
        B[Lifecycle Manager]
        C((Event Bus: Tokio Broadcast))
        D[Task Scheduler]
        E[State Manager]
        
        B --> C
        D --> C
        E --> C
    end

    subgraph Out-of-Process Agents
        F[Plugin Sandbox WSB]
        G[Background Learner]
    end

    A <==>|gRPC / Named Pipes| C
    C <==>|Named Pipes| F
    C <==>|Named Pipes| G
    B -->|windows-rs| H[Windows OS APIs]


4. Interfaces

IVoxyEventBus: publish(topic: String, payload: Bytes), subscribe(topic: String) -> Receiver.

ILifecycle: start_module(id), stop_module(id), health_check(id).

IPC Protocol: Protocol Buffers over local Named Pipes (\\.\pipe\voxy_core).

5. Data Flow (Startup Sequence)

OS launches voxy_daemon.exe on user login.

Kernel initializes Tokio runtime.

Event Bus mounts.

Lifecycle Manager spawns critical threads: Audio (Critical), Memory DB (Normal), UI gRPC Server (Normal).

Kernel broadcasts SystemState::Ready.

6. Failure Modes

Event Bus Congestion: Slow consumers cause memory bloat.

Deadlocks: Cross-module synchronous calls locking the async runtime.

OS Suspension: Windows Sleep state tearing down hardware handles.

7. Recovery Strategy

Congestion: Drop non-critical events (e.g., telemetry) if channel capacity exceeds 10,000.

Deadlocks: Strict prohibition of blocking calls in async contexts. Use tokio::task::spawn_blocking for UIA/Win32 interop.

Suspension: Listen for PBT_APMSUSPEND to gracefully flush SQLite DBs, and PBT_APMRESUMEAUTOMATIC to reinitialize WASAPI handles.

8. Trade-offs

Rust + Named Pipes vs. Monolithic C++: Adds serialization overhead for IPC, but isolates agent crashes from bringing down the core voice loop (crucial for perceived reliability).

9. Future Extension Points

Distributed event bus allowing a local VOXY kernel to securely route automation events to other Windows devices on the same local network.