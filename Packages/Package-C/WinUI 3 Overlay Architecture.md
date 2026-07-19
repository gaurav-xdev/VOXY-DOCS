SECTION 11.1: WinUI 3 Overlay & System Tray Architecture

1. Purpose

To provide VOXY with a lightweight, high-performance visual interface that maintains sub-frame latency, ensuring that visual feedback (the "VOXY Orb") stays synchronized with audio synthesis.

2. Responsibilities

Desktop Overlay: A transparent, top-level window (DirectX-accelerated) that renders the VOXY visual avatar/state.

System Tray Application: A persistent background runner handling app lifecycle, status icons, and quick-access settings.

IPC Bridge: A gRPC-based communication layer to the Rust Kernel for asynchronous data flow.

3. Architecture

A. Communication Pattern

We utilize gRPC over Unix Domain Sockets (or Named Pipes on Windows). This keeps the UI process decoupled from the Rust kernel—if the UI crashes due to a GDI/DirectX issue, the Voice Agent and Desktop Agent continue to function seamlessly in the background.

graph LR
    subgraph VOXY Kernel (Rust)
        A[Event Bus]
    end

    subgraph UI Process (WinUI 3 / C#)
        B[gRPC Server]
        C[XAML Rendering Engine]
    end

    A <-->|Named Pipe / gRPC| B
    B -->|State Updates| C


4. Interfaces

VoxyUiService: StreamVisualState(), NotifySystemEvent(event_type: String).

5. Performance Goals

Startup: < 500ms to system tray.

Idle RAM: < 30MB.

Rendering: Consistent 60FPS when the VOXY orb is animated.