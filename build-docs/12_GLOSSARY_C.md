# VOXY Glossary — C

| Field | Value |
|-------|-------|
| **Version** | 1.0.0 |
| **Status** | Production-Ready |
| **Last Updated** | 2026-07-17 |
| **Author** | VOXY Engineering Team |
| **Classification** | Internal — Terminology Foundation |

---

## CPAL (Cross-Platform Audio Library)

| Field | Value |
|-------|-------|
| **Category** | Technology / Audio |
| **Definition** | A low-level audio input and output library for Rust that abstracts platform-specific audio APIs. On Windows, CPAL uses WASAPI as its primary backend with optional ASIO support. |
| **Purpose** | Provides a unified, safe Rust API for audio capture and playback across platforms while leveraging native low-latency audio APIs on Windows. |
| **Used By** | MOD-AUDIO |
| **Related Terms** | WASAPI, ASIO, Audio Backend, Sample Rate |
| **Related Modules** | MOD-AUDIO |
| **Related Specifications** | [03_TECH_STACK.md](03_TECH_STACK.md) |
| **Architecture Notes** | CPAL is configured for event-driven mode on Windows to minimize latency. The audio thread runs at high priority to prevent dropouts. VOXY uses CPAL's stream callback API for real-time processing. |

---

## Callback

| Field | Value |
|-------|-------|
| **Category** | Programming / Pattern |
| **Definition** | A function or closure passed as an argument to another function, to be invoked when a specific event occurs. In VOXY, callbacks are used for audio stream processing, event handling, and async completion notifications. |
| **Purpose** | Enables event-driven programming patterns where components react to external events without polling. |
| **Used By** | MOD-AUDIO, MOD-EVENT, MOD-UI |
| **Related Terms** | Closure, Handler, Listener, Delegate |
| **Related Modules** | MOD-AUDIO, MOD-EVENT |
| **Related Specifications** | [04_CODING_STANDARDS.md](04_CODING_STANDARDS.md) |
| **Architecture Notes** | VOXY prefers async/await over callbacks for new code. Where callbacks are necessary (e.g., CPAL audio streams), they are wrapped in async channels to bridge sync and async contexts safely. |

---

## Cancellation Token

| Field | Value |
|-------|-------|
| **Category** | Async / Concurrency |
| **Definition** | A mechanism for requesting cooperative cancellation of asynchronous operations. Tokio's CancellationToken allows tasks to check for cancellation requests and shut down gracefully. |
| **Purpose** | Enables clean shutdown of long-running async operations (audio capture, model inference) without forceful termination. |
| **Used By** | MOD-AUDIO, MOD-ASR, MOD-LLM, MOD-ACTION |
| **Related Terms** | Task Cancellation, Graceful Shutdown, Abort |
| **Related Modules** | All async modules |
| **Related Specifications** | [04_CODING_STANDARDS.md](04_CODING_STANDARDS.md) |
| **Architecture Notes** | Every async task in VOXY accepts a CancellationToken. Tasks check token.is_cancelled() at safe points and perform cleanup before returning. This prevents resource leaks and ensures deterministic shutdown. |

---

## Channel

| Field | Value |
|-------|-------|
| **Category** | Async / Concurrency |
| **Definition** | A communication primitive for passing messages between asynchronous tasks. Tokio provides bounded and unbounded channels (mpsc, oneshot, broadcast) for different communication patterns. |
| **Purpose** | Enables safe data transfer between concurrent components without shared mutable state. |
| **Used By** | All async modules |
| **Related Terms** | MPSC, Broadcast, Message Passing, Queue |
| **Related Modules** | MOD-EVENT, MOD-AUDIO |
| **Related Specifications** | [04_CODING_STANDARDS.md](04_CODING_STANDARDS.md) |
| **Architecture Notes** | VOXY uses bounded channels with backpressure for all inter-module communication. Channel capacities are tuned per module: audio streams use larger buffers (1024-4096), control signals use smaller buffers (16-64). |

---

## Circuit Breaker

| Field | Value |
|-------|-------|
| **Category** | Reliability / Pattern |
| **Definition** | A design pattern that prevents cascading failures by stopping requests to a failing service and returning a fallback response. After a timeout, the circuit breaker allows a test request to check if the service has recovered. |
| **Purpose** | Prevents resource exhaustion and cascading failures when external services (or internal components) become unresponsive. |
| **Used By** | MOD-ACTION, MOD-UPDATE, MOD-WINAPI |
| **Related Terms** | Fallback, Retry, Timeout, Resilience |
| **Related Modules** | MOD-ACTION |
| **Related Specifications** | [05_ENGINEERING_RULES.md](05_ENGINEERING_RULES.md) (REL-003) |
| **Architecture Notes** | VOXY implements circuit breakers for all Windows API calls and plugin invocations. State transitions: Closed (normal) -> Open (failure threshold exceeded) -> Half-Open (test request) -> Closed (success) or Open (failure). |

---

## COM (Component Object Model)

| Field | Value |
|-------|-------|
| **Category** | Technology / Windows |
| **Definition** | Microsoft's binary-interface standard for software components on Windows. COM enables inter-process communication and dynamic object creation across different programming languages. |
| **Purpose** | Enables VOXY to interact with Windows system services, audio APIs, and UI automation that expose COM interfaces. |
| **Used By** | MOD-WINAPI, MOD-UIAUTO, MOD-APPCTL |
| **Related Terms** | WinRT, Interface, HRESULT, CoCreateInstance |
| **Related Modules** | MOD-WINAPI, MOD-UIAUTO |
| **Related Specifications** | [37_WINDOWS_API_GUIDE.md](37_WINDOWS_API_GUIDE.md) |
| **Architecture Notes** | VOXY uses windows-rs for safe COM interop from Rust. COM objects are managed via wil::com_ptr in C++ and through RAII wrappers in Rust. All COM initialization follows apartment-threading rules. |

---

## Context Window

| Field | Value |
|-------|-------|
| **Category** | AI / LLM |
| **Definition** | The maximum number of tokens an LLM can process in a single forward pass, including both input prompt and generated output. VOXY targets 8K-32K context windows for local models. |
| **Purpose** | Determines how much conversation history and background information the LLM can retain for coherent multi-turn interactions. |
| **Used By** | MOD-LLM, MOD-NLU |
| **Related Terms** | Token, KV Cache, Attention, Sliding Window |
| **Related Modules** | MOD-LLM |
| **Related Specifications** | [03_TECH_STACK.md](03_TECH_STACK.md), [09_IMPLEMENTATION_CHECKLISTS.md](09_IMPLEMENTATION_CHECKLISTS.md) |
| **Architecture Notes** | VOXY implements sliding window attention and KV cache eviction to handle contexts exceeding the model's nominal window. When the context grows too large, older turns are summarized and the KV cache is pruned to maintain performance. |

---

*End of 12_GLOSSARY_C.md*
