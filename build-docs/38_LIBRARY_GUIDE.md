# VOXY — Library Guide

| Field | Value |
|-------|-------|
| **Version** | 1.0.0 |
| **Status** | Production-Ready |

---

## Core Library Registry

| Library | Version | Purpose | Module | Notes |
|---------|---------|---------|--------|-------|
| tokio | 1.40+ | Async runtime | All | Full feature set |
| serde | 1.0+ | Serialization | All | derive feature |
| tracing | 0.1+ | Logging | All | json feature for CI |
| ort | 2.0+ | ONNX Runtime | MOD-ASR, MOD-LLM | directml feature |
| cpal | 0.15+ | Audio I/O | MOD-AUDIO | wasapi default |
| windows | 0.58+ | Win32/WinRT | MOD-WINAPI | selective features |
| rusqlite | 0.32+ | Embedded DB | MOD-CONFIG, MOD-KB | bundled feature |

## Integration Patterns

### Feature Flags

```toml
[features]
default = ["wasapi"]
wasapi = []
asio = ["cpal/asio"]
directml = ["ort/directml"]
```

### Version Pinning

All versions pinned in workspace Cargo.toml. No wildcard or caret-only versions.

---

*End of 38_LIBRARY_GUIDE.md*