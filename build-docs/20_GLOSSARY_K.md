# VOXY Glossary — K

| Field | Value |
|-------|-------|
| **Version** | 1.0.0 |
| **Status** | Production-Ready |
| **Last Updated** | 2026-07-17 |
| **Author** | VOXY Engineering Team |
| **Classification** | Internal — Terminology Foundation |

---

## KV Cache (Key-Value Cache)

| Field | Value |
|-------|-------|
| **Category** | AI / LLM |
| **Definition** | A memory structure that stores the key and value tensors from previous tokens in transformer attention layers, avoiding redundant computation during autoregressive generation. |
| **Purpose** | Dramatically speeds up LLM inference by reusing attention computations from prior tokens, reducing complexity from O(n^2) to O(n) per new token. |
| **Used By** | MOD-LLM |
| **Related Terms** | Attention, Transformer, Token, Context Window |
| **Related Modules** | MOD-LLM |
| **Related Specifications** | [03_TECH_STACK.md](03_TECH_STACK.md), [09_IMPLEMENTATION_CHECKLISTS.md](09_IMPLEMENTATION_CHECKLISTS.md) |
| **Architecture Notes** | VOXY implements KV cache with memory-mapped storage for large contexts. Cache eviction uses a sliding window strategy: when the context exceeds the window size, oldest KV pairs are discarded. Memory usage is monitored to prevent OOM. |

---

## Knowledge Base

| Field | Value |
|-------|-------|
| **Category** | Component / AI |
| **Definition** | A persistent store of structured and unstructured information that the AI can query to answer questions and inform responses. VOXY's Knowledge Base uses vector embeddings for semantic search. |
| **Purpose** | Extends the LLM's knowledge beyond its training data with user-specific documents, system documentation, and reference materials. |
| **Used By** | MOD-KB, MOD-LLM |
| **Related Terms** | Vector Store, RAG, Embedding, Document Retrieval |
| **Related Modules** | MOD-KB |
| **Related Specifications** | [01_PROJECT_STRUCTURE.md](01_PROJECT_STRUCTURE.md), [09_IMPLEMENTATION_CHECKLISTS.md](09_IMPLEMENTATION_CHECKLISTS.md) |
| **Architecture Notes** | VOXY's Knowledge Base stores documents as chunks with embeddings in an HNSW index. Documents are ingested from user-specified folders and indexed incrementally. Retrieval combines semantic search (embedding similarity) with keyword search (BM25) for hybrid results. |

---

## Keyword Spotting

| Field | Value |
|-------|-------|
| **Category** | AI / Audio |
| **Definition** | A specialized form of speech recognition that detects predefined keywords or phrases in an audio stream. Wake word detection is a specific application of keyword spotting. |
| **Purpose** | Enables always-on listening for activation commands with minimal resource usage. |
| **Used By** | MOD-WAKE |
| **Related Terms** | Wake Word, Hotword, Detection, Trigger |
| **Related Modules** | MOD-WAKE |
| **Related Specifications** | [09_IMPLEMENTATION_CHECKLISTS.md](09_IMPLEMENTATION_CHECKLISTS.md) |
| **Architecture Notes** | VOXY's keyword spotting uses a small ONNX model (<5MB) running continuously on a low-priority thread. The model processes 20ms audio frames and outputs a confidence score. Detection triggers the full ASR pipeline. |

---

## Kernel

| Field | Value |
|-------|-------|
| **Category** | AI / Compute |
| **Definition** | A function or operation in a computational graph, particularly in neural networks. Kernels implement specific mathematical operations (matrix multiplication, convolution, attention) optimized for target hardware. |
| **Purpose** | Provides the fundamental compute operations that compose AI model inference. |
| **Used By** | MOD-ASR, MOD-LLM, MOD-NLU |
| **Related Terms** | Operator, CUDA Kernel, Shader, Graph Node |
| **Related Modules** | MOD-LLM |
| **Related Specifications** | [03_TECH_STACK.md](03_TECH_STACK.md) |
| **Architecture Notes** | ONNX Runtime selects optimized kernels based on the execution provider (DirectML shaders for GPU, MKL-DNN for CPU). VOXY benefits from kernel fusion, where multiple operations are combined into a single kernel for reduced memory bandwidth. |

---

*End of 20_GLOSSARY_K.md*