# VOXY Glossary — H

| Field | Value |
|-------|-------|
| **Version** | 1.0.0 |
| **Status** | Production-Ready |
| **Last Updated** | 2026-07-17 |
| **Author** | VOXY Engineering Team |
| **Classification** | Internal — Terminology Foundation |

---

## HNSW (Hierarchical Navigable Small World)

| Field | Value |
|-------|-------|
| **Category** | AI / Algorithm |
| **Definition** | A graph-based approximate nearest neighbor search algorithm that constructs a multi-layer navigable graph for efficient similarity search in high-dimensional spaces. |
| **Purpose** | Enables fast semantic search over document embeddings in the Knowledge Base with sub-millisecond query times. |
| **Used By** | MOD-KB |
| **Related Terms** | Vector Search, ANN, Embedding, Similarity |
| **Related Modules** | MOD-KB |
| **Related Specifications** | [03_TECH_STACK.md](03_TECH_STACK.md) |
| **Architecture Notes** | VOXY uses an HNSW index (via the hnsw crate) for vector search over document embeddings. The index is persisted to disk and loaded at startup. Parameters: M=16, ef_construction=200, ef_search=64. Index rebuilds are incremental. |

---

## HRESULT

| Field | Value |
|-------|-------|
| **Category** | Windows / Error Handling |
| **Definition** | A 32-bit error code used by Windows COM interfaces to indicate success or failure. Values >= 0 indicate success; negative values indicate errors. |
| **Purpose** | Standardizes error reporting across Windows APIs, enabling consistent error handling in VOXY's Windows integration layer. |
| **Used By** | MOD-WINAPI, MOD-UIAUTO, MOD-APPCTL |
| **Related Terms** | COM, Win32 Error, NTSTATUS, Error Code |
| **Related Modules** | MOD-WINAPI |
| **Related Specifications** | [37_WINDOWS_API_GUIDE.md](37_WINDOWS_API_GUIDE.md) |
| **Architecture Notes** | VOXY wraps HRESULT values in Rust Result types using the windows-rs crate's Error type. Common HRESULTs: S_OK (0), E_FAIL (0x80004005), E_INVALIDARG (0x80070057). All HRESULT returns are checked and converted to domain-specific errors. |

---

## Hotfix

| Field | Value |
|-------|-------|
| **Category** | Process / Release |
| **Definition** | An urgent patch released outside the normal release cycle to fix a critical bug or security vulnerability in production. |
| **Purpose** | Minimizes the impact of critical issues by delivering fixes rapidly without waiting for the next scheduled release. |
| **Used By** | Release Engineering, DevOps |
| **Related Terms** | Patch, Emergency Release, Critical Fix |
| **Related Modules** | MOD-UPDATE |
| **Related Specifications** | [06_DEVELOPER_WORKFLOW.md](06_DEVELOPER_WORKFLOW.md), [40_RELEASE_CHECKLIST.md](40_RELEASE_CHECKLIST.md) |
| **Architecture Notes** | VOXY hotfixes follow an expedited workflow: branch from main, minimal fix, 1-approver review, immediate merge, and deployment. Post-incident reviews are conducted within 48 hours. Hotfix branches are named hotfix/{version}. |

---

## Handler

| Field | Value |
|-------|-------|
| **Category** | Architecture / Pattern |
| **Definition** | A component that processes specific events or requests. In VOXY, handlers process events from the Event Bus, user commands from the Action Engine, and window messages from Windows. |
| **Purpose** | Encapsulates the processing logic for specific events, enabling modular and testable event-driven architecture. |
| **Used By** | MOD-EVENT, MOD-ACTION, MOD-UI |
| **Related Terms** | Listener, Processor, Callback, Controller |
| **Related Modules** | MOD-EVENT, MOD-ACTION |
| **Related Specifications** | [01_PROJECT_STRUCTURE.md](01_PROJECT_STRUCTURE.md) |
| **Architecture Notes** | VOXY handlers implement the Handler trait with a handle method. Handlers are registered with the Event Bus or Action Engine and receive typed events. Each handler runs in its own async task for isolation. |

---

## Hardware Abstraction

| Field | Value |
|-------|-------|
| **Category** | Architecture / Design |
| **Definition** | A layer of software that hides hardware-specific details from higher-level code, providing a uniform interface across different hardware configurations. |
| **Purpose** | Enables VOXY to run on diverse hardware (different GPUs, audio devices, CPUs) without code changes. |
| **Used By** | MOD-AUDIO, MOD-LLM, MOD-WINAPI |
| **Related Terms** | HAL, Abstraction Layer, Portability, Backend |
| **Related Modules** | MOD-AUDIO, MOD-WINAPI |
| **Related Specifications** | [03_TECH_STACK.md](03_TECH_STACK.md) |
| **Architecture Notes** | VOXY abstracts audio hardware via CPAL (WASAPI/ASIO backends) and AI acceleration via ONNX Runtime (DirectML/CPU execution providers). The abstraction layers are implemented as traits with backend-specific implementations. |

---

*End of 17_GLOSSARY_H.md*