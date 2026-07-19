# VOXY Glossary — M

| Field | Value |
|-------|-------|
| **Version** | 1.0.0 |
| **Status** | Production-Ready |
| **Last Updated** | 2026-07-17 |
| **Author** | VOXY Engineering Team |
| **Classification** | Internal — Terminology Foundation |

---

## MSIX

| Field | Value |
|-------|-------|
| **Category** | Technology / Deployment |
| **Definition** | Microsoft's modern Windows app packaging format that provides a clean installation, update, and uninstall experience. MSIX packages are containerized and support automatic updates via the Microsoft Store or App Installer. |
| **Purpose** | Distributes VOXY as a trusted, sandboxed Windows application with clean installation and reliable updates. |
| **Used By** | MOD-UPDATE, MOD-UI |
| **Related Terms** | App Installer, Windows Store, Package, Manifest |
| **Related Modules** | MOD-UPDATE |
| **Related Specifications** | [02_BUILD_ORDER.md](02_BUILD_ORDER.md), [40_RELEASE_CHECKLIST.md](40_RELEASE_CHECKLIST.md) |
| **Architecture Notes** | VOXY's MSIX package includes the main executable, Rust DLLs, model files, and assets. The package is signed with a code signing certificate for SmartScreen compatibility. Updates are delivered via App Installer with differential downloads. |

---

## MVVM (Model-View-ViewModel)

| Field | Value |
|-------|-------|
| **Category** | Architecture / UI Pattern |
| **Definition** | A UI design pattern that separates the user interface (View) from the business logic (ViewModel) and data (Model). The ViewModel exposes data and commands to the View via data binding. |
| **Purpose** | Enables testable UI logic, separation of concerns, and responsive UI updates without code-behind complexity. |
| **Used By** | MOD-UI |
| **Related Terms** | Data Binding, ViewModel, XAML, INotifyPropertyChanged |
| **Related Modules** | MOD-UI |
| **Related Specifications** | [04_CODING_STANDARDS.md](04_CODING_STANDARDS.md) |
| **Architecture Notes** | VOXY's WinUI 3 frontend uses MVVM with CommunityToolkit.Mvvm for source generators. ViewModels communicate with the Rust backend via IPC services. The pattern ensures that UI logic is unit-testable without rendering actual UI elements. |

---

## Model Quantization

| Field | Value |
|-------|-------|
| **Category** | AI / Optimization |
| **Definition** | The process of reducing the precision of model weights and activations (e.g., from 32-bit floats to 8-bit integers) to decrease model size and increase inference speed. |
| **Purpose** | Enables large AI models to run on consumer hardware with limited RAM and CPU/GPU resources. |
| **Used By** | MOD-ASR, MOD-LLM, MOD-NLU, MOD-WAKE, MOD-TTS |
| **Related Terms** | INT8, Q4, Q5, ONNX, Calibration, Compression |
| **Related Modules** | MOD-LLM, MOD-ASR |
| **Related Specifications** | [03_TECH_STACK.md](03_TECH_STACK.md) |
| **Architecture Notes** | VOXY uses multiple quantization schemes: QDQ (Quantize-Dequantize) for ONNX Runtime INT8, GGUF Q4/Q5 for Candle LLMs, and custom quantization for wake word models. Each model is validated post-quantization to ensure accuracy remains within acceptable bounds. |

---

## Mutex

| Field | Value |
|-------|-------|
| **Category** | Concurrency / Programming |
| **Definition** | A synchronization primitive that ensures mutual exclusion — only one thread can hold the lock at a time, preventing concurrent access to shared data. |
| **Purpose** | Protects shared state from race conditions in multi-threaded environments. |
| **Used By** | All modules |
| **Related Terms** | Lock, RwLock, Semaphore, Critical Section |
| **Related Modules** | All modules |
| **Related Specifications** | [04_CODING_STANDARDS.md](04_CODING_STANDARDS.md) |
| **Architecture Notes** | VOXY prefers parking_lot::Mutex over std::sync::Mutex for better performance and const constructor support. Mutexes are avoided in hot paths in favor of lock-free structures. |

---

## Memory Mapping

| Field | Value |
|-------|-------|
| **Category** | OS / Performance |
| **Definition** | A mechanism that maps a file or device into the virtual memory address space of a process, allowing file contents to be accessed as if they were in memory. |
| **Purpose** | Enables efficient loading of large model files without copying data into process memory, reducing RAM usage and load times. |
| **Used By** | MOD-LLM, MOD-ASR, MOD-AUDIO |
| **Related Terms** | mmap, memmap2, Virtual Memory, Page Fault |
| **Related Modules** | MOD-LLM |
| **Related Specifications** | [03_TECH_STACK.md](03_TECH_STACK.md) |
| **Architecture Notes** | VOXY uses memmap2 for memory-mapping model weight files. This allows the OS to page data in on demand and evict unused pages under memory pressure. Memory-mapped models load 10x faster than read() for large files. |

---

*End of 22_GLOSSARY_M.md*