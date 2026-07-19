# VOXY — Coding Standards

| Field | Value |
|-------|-------|
| **Version** | 1.0.0 |
| **Status** | Production-Ready |
| **Last Updated** | 2026-07-17 |
| **Author** | VOXY Engineering Team |
| **Classification** | Internal — Quality Foundation |

---

## Purpose

This document defines the coding conventions, style rules, architectural patterns, and quality gates for all code written in the VOXY codebase. It ensures consistency, readability, maintainability, and safety across all modules and contributors.

---

## Scope

Covers:
- Rust coding standards (primary language)
- C/C++ interop standards
- C# / WinUI 3 standards
- Python scripting standards
- Code formatting and linting rules
- Documentation requirements
- Testing standards
- Error handling patterns
- Async/await conventions
- Unsafe code guidelines

Does not cover:
- Technology selections (see [03_TECH_STACK.md](03_TECH_STACK.md))
- Module architecture (see [01_PROJECT_STRUCTURE.md](01_PROJECT_STRUCTURE.md))
- Git workflow (see [06_DEVELOPER_WORKFLOW.md](06_DEVELOPER_WORKFLOW.md))

---

## Audience

- All software engineers contributing to VOXY
- AI Coding Agents generating code
- Code reviewers conducting PR reviews
- CI/CD pipeline authors

---

## Rust Coding Standards

### General Principles

1. **Zero warnings policy.** `cargo clippy -- -D warnings` must pass.
2. **No `unwrap()` or `expect()` in production code.** Use `?` propagation or explicit error handling.
3. **No `unsafe` without ADR.** Every `unsafe` block requires an Architecture Decision Record.
4. **Async-first.** All I/O operations must be async using Tokio.
5. **Explicit types for public APIs.** Public function signatures must not rely on type inference.

### Formatting

- Use `rustfmt` with default settings.
- Max line length: 100 characters.
- Tab width: 4 spaces.
- Import grouping: std, external crates, internal modules.

```rust
// Good
use std::sync::Arc;

use tokio::sync::mpsc;
use tracing::{info, warn};

use crate::event_bus::{Event, EventBus};
use crate::config::Settings;

// Bad — mixed grouping
use tokio::sync::mpsc;
use std::sync::Arc;
use crate::config::Settings;
use tracing::info;
```

### Naming Conventions

| Item | Convention | Example |
|------|-----------|---------|
| Modules | `snake_case` | `audio_pipeline` |
| Types / Structs / Enums | `PascalCase` | `AudioCaptureEngine` |
| Traits | `PascalCase` (adjective-like) | `Capturable`, `Playable` |
| Functions / Methods | `snake_case` | `start_capture()` |
| Variables | `snake_case` | `sample_rate` |
| Constants | `SCREAMING_SNAKE_CASE` | `MAX_BUFFER_SIZE` |
| Static variables | `SCREAMING_SNAKE_CASE` | `GLOBAL_CONFIG` |
| Type parameters | `PascalCase`, single letter preferred | `T`, `E`, `Ctx` |
| Lifetimes | `snake_case`, single letter | `'a`, `'ctx` |
| Feature flags | `snake_case` | `directml` |
| File names | `snake_case.rs` | `audio_capture.rs` |

### Error Handling

```rust
// Use thiserror for library errors
#[derive(thiserror::Error, Debug)]
pub enum AudioError {
    #[error("device not found: {0}")]
    DeviceNotFound(String),

    #[error("invalid sample rate: {expected}, got {actual}")]
    InvalidSampleRate { expected: u32, actual: u32 },

    #[error("IO error: {0}")]
    Io(#[from] std::io::Error),
}

// Use anyhow for application errors
use anyhow::{Context, Result};

fn load_model(path: &Path) -> Result<Model> {
    let data = std::fs::read(path)
        .with_context(|| format!("failed to read model from {}", path.display()))?;
    Model::deserialize(&data)
        .context("failed to deserialize model")
}
```

### Async Patterns

```rust
// Prefer spawn for CPU-bound work
let result = tokio::task::spawn_blocking(|| {
    heavy_computation(input)
}).await?;

// Use bounded channels for backpressure
let (tx, mut rx) = tokio::sync::mpsc::channel::<AudioFrame>(1024);

// Always handle cancellation
async fn process_stream(mut rx: mpsc::Receiver<AudioFrame>) {
    while let Some(frame) = rx.recv().await {
        if tokio::task::yield_now().is_err() {
            break; // Task cancelled
        }
        process_frame(frame).await;
    }
}
```

### Unsafe Code Guidelines

```rust
// Every unsafe block MUST have a SAFETY comment
// SAFETY: The pointer is valid because it was allocated by `Box::into_raw`
// and has not been freed. The caller ensures the pointer is non-null.
unsafe { 
    std::ptr::read(ptr) 
}

// Prefer safe abstractions
// BAD
unsafe { *ptr = value; }

// GOOD
ptr.write(value); // Safe method on raw pointer
```

### Documentation Standards

Every public item must have doc comments:

```rust
/// Captures audio from the default input device.
///
/// # Arguments
///
/// * `config` — The desired stream configuration.
/// * `callback` — Called for each captured audio buffer.
///
/// # Errors
///
/// Returns `AudioError::DeviceNotFound` if no input device is available.
/// Returns `AudioError::InvalidSampleRate` if the requested rate is unsupported.
///
/// # Examples
///
/// ```no_run
/// let engine = AudioCaptureEngine::new()?;
/// let stream = engine.capture_default(Default::default(), |data| {
///     println!("Captured {} samples", data.len());
/// })?;
/// ```
pub fn capture_default(
    &self,
    config: StreamConfig,
    callback: impl FnMut(&[f32]) + Send + 'static,
) -> Result<AudioStream, AudioError> {
    // ...
}
```

---

## C/C++ Coding Standards

### General Principles

1. **Modern C++ (C++20).** Use `std::unique_ptr`, `std::span`, `std::optional`.
2. **RAII everywhere.** No raw `new`/`delete`; use smart pointers.
3. **COM safety.** Always check `HRESULT`; use `wil::com_ptr` for COM objects.
4. **Exception safety.** Use `noexcept` where appropriate; document exception guarantees.

### Naming Conventions

| Item | Convention | Example |
|------|-----------|---------|
| Classes / Structs | `PascalCase` | `ComInteropHelper` |
| Functions | `PascalCase` | `InitializeAudioClient` |
| Variables | `camelCase` | `sampleRate` |
| Member variables | `m_camelCase` | `m_audioClient` |
| Constants | `kPascalCase` | `kMaxBufferSize` |
| Macros | `SCREAMING_SNAKE_CASE` | `VOXY_ASSERT` |
| Namespaces | `PascalCase` | `Voxy::Win32` |

### COM Interop Pattern

```cpp
#include <wil/com.h>
#include <windows.h>

HRESULT InitializeComAudio(
    wil::com_ptr<IAudioClient>& audioClient,
    const WAVEFORMATEX& format
) noexcept {
    wil::com_ptr<IMMDeviceEnumerator> enumerator;
    HRESULT hr = CoCreateInstance(
        __uuidof(MMDeviceEnumerator),
        nullptr,
        CLSCTX_ALL,
        IID_PPV_ARGS(&enumerator)
    );
    if (FAILED(hr)) {
        return hr;
    }

    wil::com_ptr<IMMDevice> device;
    hr = enumerator->GetDefaultAudioEndpoint(eCapture, eConsole, &device);
    if (FAILED(hr)) {
        return hr;
    }

    hr = device->Activate(
        __uuidof(IAudioClient),
        CLSCTX_ALL,
        nullptr,
        reinterpret_cast<void**>(audioClient.put())
    );
    return hr;
}
```

---

## C# / WinUI 3 Coding Standards

### General Principles

1. **MVVM pattern.** All UI logic in ViewModels; code-behind only for view-specific behavior.
2. **Async/await everywhere.** No blocking calls on UI thread.
3. **Null safety.** Use nullable reference types (`<Nullable>enable</Nullable>`).
4. **Dependency injection.** Use `Microsoft.Extensions.DependencyInjection`.

### Naming Conventions

| Item | Convention | Example |
|------|-----------|---------|
| Classes / Structs | `PascalCase` | `MainViewModel` |
| Interfaces | `IPascalCase` | `IAudioService` |
| Methods | `PascalCase` | `StartRecordingAsync` |
| Properties | `PascalCase` | `IsListening` |
| Fields (private) | `_camelCase` | `_audioEngine` |
| Constants | `PascalCase` | `MaxRetryCount` |
| Events | `PascalCase` + `EventHandler` suffix | `VoiceCommandReceived` |
| XAML files | `PascalCase.xaml` | `MainWindow.xaml` |

### MVVM Pattern

```csharp
// ViewModel
public partial class MainViewModel : ObservableObject
{
    private readonly IAudioService _audioService;

    [ObservableProperty]
    [NotifyPropertyChangedFor(nameof(CanStartListening))]
    private bool _isListening;

    public bool CanStartListening => !IsListening;

    [RelayCommand]
    private async Task StartListeningAsync()
    {
        IsListening = true;
        try
        {
            await _audioService.StartCaptureAsync();
        }
        finally
        {
            IsListening = false;
        }
    }
}

// View (XAML)
// <Button Content="Listen"
//         Command="{x:Bind ViewModel.StartListeningCommand}"
//         IsEnabled="{x:Bind ViewModel.CanStartListening, Mode=OneWay}" />
```

---

## Python Scripting Standards

### General Principles

1. **Type hints everywhere.** Use `from __future__ import annotations`.
2. **Ruff for linting and formatting.** Replace flake8, black, isort.
3. **No global state.** All functions receive dependencies as parameters.

### Naming Conventions

| Item | Convention | Example |
|------|-----------|---------|
| Modules | `snake_case` | `model_converter` |
| Classes | `PascalCase` | `OnnxQuantizer` |
| Functions | `snake_case` | `quantize_model` |
| Constants | `SCREAMING_SNAKE_CASE` | `DEFAULT_OPSET` |
| Variables | `snake_case` | `input_shape` |

---

## Testing Standards

### Unit Tests (Rust)

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn capture_engine_initializes_with_default_device() {
        let engine = AudioCaptureEngine::new().unwrap();
        let config = StreamConfig::default();
        let result = engine.capture_default(config, |_data| {});
        assert!(result.is_ok());
    }

    #[test]
    fn invalid_sample_rate_returns_error() {
        let engine = AudioCaptureEngine::new().unwrap();
        let mut config = StreamConfig::default();
        config.sample_rate = 999_999; // Invalid
        let result = engine.capture_default(config, |_data| {});
        assert!(matches!(result, Err(AudioError::InvalidSampleRate { .. })));
    }

    #[tokio::test]
    async fn event_bus_delivers_messages() {
        let bus = EventBus::new();
        let mut rx = bus.subscribe::<TestEvent>();
        bus.publish(TestEvent { value: 42 }).await;
        let event = rx.recv().await.unwrap();
        assert_eq!(event.value, 42);
    }
}
```

### Property-Based Tests

```rust
use proptest::prelude::*;

proptest! {
    #[test]
    fn resampling_preserves_energy(input in prop::collection::vec(-1.0f32..=1.0, 1024..=4096)) {
        let resampled = resample(&input, 44100, 16000);
        let input_energy: f32 = input.iter().map(|x| x * x).sum();
        let output_energy: f32 = resampled.iter().map(|x| x * x).sum();
        let ratio = output_energy / input_energy;
        prop_assert!((ratio - 1.0).abs() < 0.1);
    }
}
```

### Benchmarks

```rust
use criterion::{black_box, criterion_group, criterion_main, Criterion};

fn benchmark_asr_inference(c: &mut Criterion) {
    let model = load_test_model();
    let audio = load_test_audio();

    c.bench_function("asr_inference_5s", |b| {
        b.iter(|| model.transcribe(black_box(&audio)))
    });
}

criterion_group!(benches, benchmark_asr_inference);
criterion_main!(benches);
```

---

## Code Review Checklist

- [ ] Code follows naming conventions for the language.
- [ ] No `unwrap()` or `expect()` in production paths.
- [ ] All public items have doc comments.
- [ ] Error types are specific and implement `std::error::Error`.
- [ ] Async code handles cancellation.
- [ ] Unsafe code has SAFETY comments and ADR reference.
- [ ] Tests cover happy path, error paths, and edge cases.
- [ ] Benchmarks exist for performance-critical code.
- [ ] No dead code or unused imports.
- [ ] `cargo clippy -- -D warnings` passes.
- [ ] `cargo fmt --check` passes.
- [ ] `cargo test` passes.
- [ ] `cargo audit` is clean.

---

## References

- [Rust API Guidelines](https://rust-lang.github.io/api-guidelines/)
- [Rust Style Guide](https://doc.rust-lang.org/style-guide/)
- [Effective Rust](https://www.lurklurk.org/effective-rust/)
- [Microsoft C++ Core Guidelines](https://isocpp.github.io/CppCoreGuidelines/)
- [C# Coding Conventions](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions)
- [WinUI 3 MVVM Guide](https://learn.microsoft.com/en-us/windows/apps/design/data-binding/data-binding-and-mvvm)

---

## Cross References

- See [03_TECH_STACK.md](03_TECH_STACK.md) for language and library selections.
- See [05_ENGINEERING_RULES.md](05_ENGINEERING_RULES.md) for non-negotiable constraints.
- See [06_DEVELOPER_WORKFLOW.md](06_DEVELOPER_WORKFLOW.md) for Git and review workflow.
- See [08_MODULE_TEMPLATE.md](08_MODULE_TEMPLATE.md) for module scaffolding.

---

## Best Practices

1. **Write tests first** for bug fixes (regression tests).
2. **Document public APIs** before implementation.
3. **Use `tracing` spans** for async operation tracking.
4. **Prefer composition over inheritance** in all languages.
5. **Keep functions small** (<50 lines ideal, <100 lines maximum).

---

## Common Mistakes

| Mistake | Consequence | Prevention |
|---------|-------------|------------|
| `unwrap()` in production | Panics, crashes | Clippy lint `unwrap_used` |
| Blocking in async context | Thread starvation | Always use `spawn_blocking` |
| Missing error context | Unactionable logs | Use `with_context()` |
| Raw pointers without SAFETY | Undefined behavior | Mandatory SAFETY comment + ADR |
| Public API without docs | Poor developer experience | CI gate on missing docs |

---

## Review Checklist

- [ ] All languages have documented conventions.
- [ ] Naming conventions are consistent with [26_NAMING_CONVENTIONS.md](26_NAMING_CONVENTIONS.md).
- [ ] Error handling patterns are followed.
- [ ] Async patterns are documented and enforced.
- [ ] Unsafe code guidelines are clear.
- [ ] Testing standards cover unit, integration, and property-based tests.
- [ ] Code review checklist is used for every PR.

---

*End of 04_CODING_STANDARDS.md*
