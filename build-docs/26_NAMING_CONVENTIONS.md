# VOXY — Naming Conventions

| Field | Value |
|-------|-------|
| **Version** | 1.0.0 |
| **Status** | Production-Ready |
| **Last Updated** | 2026-07-17 |

---

## Cross-Language Naming Matrix

| Concept | Rust | C++ | C# | Python | Files/Directories |
|---------|------|-----|-----|--------|-------------------|
| Module | snake_case | PascalCase | PascalCase | snake_case | snake-case |
| Type | PascalCase | PascalCase | PascalCase | PascalCase | — |
| Function | snake_case | PascalCase | PascalCase | snake_case | — |
| Variable | snake_case | camelCase | camelCase | snake_case | — |
| Constant | SCREAMING_SNAKE | kPascalCase | PascalCase | SCREAMING_SNAKE | — |
| Member | — | m_camelCase | _camelCase | — | — |
| File | snake_case.rs | PascalCase.cpp | PascalCase.cs | snake_case.py | kebab-case |

---

## Special Prefixes/Suffixes

| Prefix/Suffix | Meaning | Example |
|--------------|---------|---------|
| I prefix | Interface | IAudioService |
| Async suffix | Async method | ProcessAsync |
| Result suffix | Operation outcome type | TranscriptionResult |
| Config suffix | Configuration struct | AudioConfig |
| Error suffix | Error enum | AudioError |
| Handle suffix | Opaque resource handle | StreamHandle |

---

*End of 26_NAMING_CONVENTIONS.md*