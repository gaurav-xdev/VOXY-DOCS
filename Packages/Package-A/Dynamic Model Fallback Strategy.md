SECTION 8.3: Dynamic Model Fallback Strategy

1. Purpose

To guarantee VOXY remains highly responsive and functional across wildly different hardware configurations (from an Intel Iris integrated GPU to an RTX 4090). The fallback strategy dictates how inference seamlessly shifts between local hardware and secure cloud APIs.

2. Responsibilities

Hardware Profiling: Detect available VRAM, RAM, and Compute (DirectML/CUDA/Vulkan) on startup.

Dynamic Routing: Route tasks to the local GPU if possible, local CPU if desperate, or Cloud APIs if authorized.

Graceful Degradation: Adjust VOXY's capabilities (e.g., turning off complex Vision tasks) if hardware cannot support it, rather than crashing.

3. Architecture

A. Tiered Execution Matrix

The inference router operates on a strict fallback hierarchy.

graph TD
    A[Intent & Context Ready] --> B{Check Local Hardware Telemetry}
    
    subgraph Tier 1: Optimal Local
        B -->|Dedicated VRAM > 8GB| C[Local NPU/GPU: Llama-3 8B]
    end
    
    subgraph Tier 2: Degraded Local
        B -->|Shared VRAM / Integrated GPU| D[Local CPU/iGPU: Phi-3 4B]
    end
    
    subgraph Tier 3: Secure Cloud (Opt-in)
        B -->|Hardware Starved OR Complex Task| E[Cloud API: Groq / OpenAI]
        C -->|VRAM OOM Error| E
        D -->|Latency > 2000ms| E
    end
    
    C --> F[Execution]
    D --> F
    E --> F


B. Workload Splitting

Not all models require fallback. We split the workload:

Wake Word & VAD: ALWAYS Local CPU (Must run 24/7 without battery drain).

STT / TTS: Strongly Local (Whisper/StyleTTS2 can run on low-end hardware).

LLM (Reasoning): Subject to the Dynamic Fallback Strategy.

4. Interfaces

A. Execution Router Crate (core/src/ai/execution.rs)

pub enum InferenceBackend {
    LocalDirectML(String), // Model Path
    LocalVulkan(String),
    CloudApi(String),      // Endpoint URL
}

pub struct ExecutionRouter {
    telemetry: HardwareTelemetry,
    cloud_authorized: bool,
}

impl ExecutionRouter {
    /// Determines the fastest execution path based on real-time hardware metrics.
    pub fn get_optimal_backend(&self, required_tokens: usize) -> InferenceBackend {
        if self.telemetry.available_vram_mb() > 4000 {
            InferenceBackend::LocalDirectML("models/llama3-8b.gguf".into())
        } else if self.cloud_authorized {
            InferenceBackend::CloudApi("https://api.groq.com/v1/chat".into())
        } else {
            // Absolute fallback to slow CPU if cloud is disabled for privacy
            InferenceBackend::LocalVulkan("models/phi3-mini.gguf".into()) 
        }
    }
}


5. Data Flow (Real-time Out-of-Memory Fallback)

User requests a massive document summary.

Context Assembler generates a $7500$ token prompt.

Execution Router attempts to allocate KV-Cache on the local RTX 3050 (4GB VRAM).

llama.cpp throws an Out-Of-Memory (OOM) allocation error.

Execution Router catches the error instantly, suppresses it from the user, and re-routes the prompt payload to the configured Cloud Endpoint (e.g., Groq API).

Result is streamed back. The user experiences a $\approx 300\text{ms}$ delay but no failure.

6. Failure Modes

Privacy Violation: A user with a strict "Local Only" enterprise policy has their proprietary screen data sent to the cloud due to an automatic fallback trigger.

API Rate Limiting / Outage: The fallback cloud provider goes down or the user runs out of credits, and their local hardware is too weak to compensate.

7. Recovery Strategy

Strict Opt-In Configuration: By default, Cloud Fallback is DISABLED. The Execution Router will hard-fail and verbally inform the user: "This task requires more memory than your PC has available, and cloud processing is disabled."

Network Timeout Watchdogs: If the Cloud API doesn't return the first token within $800\text{ms}$, the router kills the connection and gracefully informs the user of a network error.

8. Trade-offs

DirectML vs. CUDA: We build VOXY atop windows-rs and DirectML by default. While CUDA is ~15% faster for Llama.cpp, DirectML ensures VOXY works on AMD, Intel ARC, and standard Windows hardware without forcing users to install massive proprietary NVIDIA toolkits.

9. Future Extension Points

Task-Specific Routing: Automatically routing easy tasks (Chat) to the local Phi-3 model, but routing complex coding/automation tasks to a remote cloud Claude 3.5 Sonnet instance.