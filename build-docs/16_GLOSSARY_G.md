# VOXY Glossary — G

| Field | Value |
|-------|-------|
| **Version** | 1.0.0 |
| **Status** | Production-Ready |
| **Last Updated** | 2026-07-17 |
| **Author** | VOXY Engineering Team |
| **Classification** | Internal — Terminology Foundation |

---

## GGUF (GPT-Generated Unified Format)

| Field | Value |
|-------|-------|
| **Category** | AI / Model Format |
| **Definition** | A binary file format for storing quantized language models, introduced by the llama.cpp project. GGUF supports multiple quantization schemes (Q4_0, Q5_K_M, Q8_0) and metadata. |
| **Purpose** | Enables efficient storage and loading of quantized LLMs with minimal memory overhead, supporting the local-AI-first requirement. |
| **Used By** | MOD-LLM, MOD-WAKE |
| **Related Terms** | Quantization, ONNX, llama.cpp, Model Weights |
| **Related Modules** | MOD-LLM |
| **Related Specifications** | [03_TECH_STACK.md](03_TECH_STACK.md) |
| **Architecture Notes** | VOXY supports GGUF models through the Candle framework as an alternative to ONNX. GGUF models are typically 4-8x smaller than FP32 equivalents with minimal quality loss. The MOD-LLM module auto-detects model format and selects the appropriate loader. |

---

## GPU Acceleration

| Field | Value |
|-------|-------|
| **Category** | Hardware / Performance |
| **Definition** | The use of a graphics processing unit (GPU) to perform general-purpose computations, particularly matrix operations for AI inference. VOXY uses GPU acceleration via DirectML. |
| **Purpose** | Reduces AI inference latency and increases throughput by leveraging parallel GPU compute capabilities. |
| **Used By** | MOD-ASR, MOD-LLM, MOD-NLU, MOD-WAKE |
| **Related Terms** | DirectML, CUDA, NPU, Inference, ONNX Runtime |
| **Related Modules** | MOD-LLM, MOD-ASR |
| **Related Specifications** | [03_TECH_STACK.md](03_TECH_STACK.md), [05_ENGINEERING_RULES.md](05_ENGINEERING_RULES.md) (PERF-004) |
| **Architecture Notes** | VOXY targets DirectML as the primary GPU acceleration path because it works across all GPU vendors without proprietary SDKs. On systems without compatible GPUs, VOXY falls back to CPU inference. GPU memory usage is monitored to prevent OOM conditions. |

---

## Graceful Degradation

| Field | Value |
|-------|-------|
| **Category** | Reliability / Architecture |
| **Definition** | The design principle that a system should continue operating at reduced functionality when components fail, rather than failing completely. |
| **Purpose** | Ensures VOXY remains usable even when AI models, audio devices, or network connectivity are unavailable. |
| **Used By** | All modules |
| **Related Terms** | Fallback, Resilience, Fault Tolerance, Circuit Breaker |
| **Related Modules** | MOD-ACTION, MOD-ASR, MOD-LLM |
| **Related Specifications** | [05_ENGINEERING_RULES.md](05_ENGINEERING_RULES.md) (REL-005) |
| **Architecture Notes** | VOXY's graceful degradation hierarchy: Full AI -> Reduced AI -> Rule-based -> Manual mode. Each level maintains core voice command functionality with decreasing sophistication. The UI communicates the current operating mode to the user. |

---

## Graph (ONNX)

| Field | Value |
|-------|-------|
| **Category** | AI / Model |
| **Definition** | A computational graph representing a neural network in the ONNX format, consisting of nodes (operations) and edges (tensors). |
| **Purpose** | Defines the structure of AI models (ASR, NLU, LLM) in a framework-agnostic format that ONNX Runtime can execute. |
| **Used By** | MOD-ASR, MOD-NLU, MOD-LLM, MOD-WAKE |
| **Related Terms** | Node, Tensor, Operator, Model |
| **Related Modules** | MOD-LLM, MOD-ASR |
| **Related Specifications** | [03_TECH_STACK.md](03_TECH_STACK.md) |
| **Architecture Notes** | VOXY models are exported to ONNX from PyTorch or TensorFlow. Graph optimization (constant folding, operator fusion) is applied at load time by ONNX Runtime. Dynamic shapes are used for variable-length inputs (audio, text). |

---

*End of 16_GLOSSARY_G.md*
