# VOXY — Windows API Guide

| Field | Value |
|-------|-------|
| **Version** | 1.0.0 |
| **Status** | Production-Ready |

---

## API Selection Matrix

| Need | Preferred API | Fallback | Notes |
|------|--------------|----------|-------|
| Audio capture | WASAPI | ASIO | Event-driven mode for low latency |
| Audio playback | WASAPI | DirectSound | Shared mode for compatibility |
| UI automation | UIA (modern) | MSAA (legacy) | UIA3 for new code |
| Window management | WinRT AppWindow | Win32 HWND | WinRT when available |
| Notifications | AppNotificationManager | ToastNotificationManager | Per Windows notifications overview |
| File operations | WinRT StorageFile | Win32 CreateFile | WinRT for sandboxed apps |
| ML inference | Windows ML | ONNX Runtime | Windows ML for in-box models |
| Crypto | Windows CNG | BCrypt/NCrypt | CNG for new code |

## COM Interop Patterns

```rust
// Use windows-rs crate
use windows::Win32::System::Com::{CoInitializeEx, COINIT_APARTMENTTHREADED};

fn initialize_com() -> Result<()> {
    unsafe {
        CoInitializeEx(None, COINIT_APARTMENTTHREADED)
            .ok()
            .context("COM initialization failed")?;
    }
    Ok(())
}
```

## Error Handling

Always check HRESULT and convert to Rust Result:

```rust
fn check_hresult(hr: HRESULT) -> Result<()> {
    if hr.is_err() {
        return Err(windows::core::Error::from_hresult(hr).into());
    }
    Ok(())
}
```

---

*End of 37_WINDOWS_API_GUIDE.md*