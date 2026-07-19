SECTION 4.3: Rebuilding OpenClaw-Style Workflows in Rust & Windows Native UIA

1. Purpose

To translate the high-level automation actions pioneered by OpenClaw (mouse clicks, visual searches, window activation) into a high-performance, deterministic C++ and Rust framework targeting Windows Native APIs directly.

2. Responsibilities

Implement Coordinate Transformation: Normalize UI spaces across multi-monitor setups with disparate DPI configurations.

Maintain High-Performance UIA Caching: Build a lightning-fast Rust cache layer for the Windows Accessibility Tree.

Provide Fallback Visual Grounding: Execute local OCR (Windows.Media.Ocr) to resolve visual targets when UIA is blocked or unsupported.

3. Architecture

A. VOXY Native Grounding Pipeline

Unlike OpenClaw's OpenCV-heavy and coordinate-brittle script execution, VOXY employs a deterministic, layered hierarchy to locate and interact with interface elements:

graph TD
    A[Agent Automation Request] --> B{Layer 1: Query UIA Cache}
    B -->|Found Element| C[Extract Native Coordinates]
    B -->|Missed / Not Supported| D{Layer 2: Trigger WinRT Capture}
    D -->|Grab Screen Buffer| E[Run Windows.Media.Ocr]
    E -->|Text Match Layout| F[Construct Coordinate Bound]
    C --> G[Normalize Coordinates]
    F --> G
    G -->|Scale with System DPI| H[Inject Input via SendInput]


B. Normalized Coordinate Modeling

To protect against DPI-induced positioning errors, VOXY translates all absolute visual targets into a normalized space $P_n = (x_n, y_n) \in [0.0, 1.0]^2$.
Before executing a hardware click event, coordinates are corrected via:


$$x_{\text{physical}} = \lfloor x_n \times \text{GetSystemMetrics}(\text{SM\_CXSCREEN}) \times \text{DPI\_scale}_x \rfloor$$

$$y_{\text{physical}} = \lfloor y_n \times \text{GetSystemMetrics}(\text{SM\_CYSCREEN}) \times \text{DPI\_scale}_y \rfloor$$

4. Interfaces

This blueprint outlines the native Rust structures communicating directly with the Windows API via windows-rs.

A. Window Manipulator Crate (core/src/automation/window.rs)

use windows::Win32::Foundation::{HWND, RECT};
use windows::Win32::UI::WindowsAndMessaging::{GetWindowRect, SetForegroundWindow, ShowWindow, SW_RESTORE};

pub struct NativeWindowManager;

impl NativeWindowManager {
    /// Forces a target application window to the foreground and restores it if minimized.
    pub fn activate_window(hwnd: HWND) -> Result<(), String> {
        unsafe {
            // Restore window if minimized
            ShowWindow(hwnd, SW_RESTORE);
            
            // Set focused active state
            if SetForegroundWindow(hwnd).as_bool() {
                Ok(())
            } else {
                Err("Failed to acquire window focus lock.".to_string())
            }
        }
    }

    /// Obtains the raw bounding box of an active HWND.
    pub fn get_window_bounds(hwnd: HWND) -> Result<RECT, String> {
        unsafe {
            let mut rect = RECT::default();
            if GetWindowRect(hwnd, &mut rect).as_bool() {
                Ok(rect)
            } else {
                Err("Could not retrieve window physical boundaries.".to_string())
            }
        }
    }
}


B. Hardware Input Driver Crate (core/src/automation/input.rs)

use std::mem::size_of;
use windows::Win32::UI::Input::KeyboardAndMouse::{
    SendInput, INPUT, INPUT_0, INPUT_MOUSE, MOUSEEVENTF_ABSOLUTE, MOUSEEVENTF_LEFTDOWN,
    MOUSEEVENTF_LEFTUP, MOUSEEVENTF_MOVE, MOUSE_INPUT,
};

pub struct MouseDriver;

impl MouseDriver {
    /// Injects a native system click at absolute normalized coordinates.
    pub fn absolute_click(x: i32, y: i32) -> Result<(), String> {
        unsafe {
            let mut inputs: [INPUT; 3] = std::mem::zeroed();

            // 1. Move to Coordinates
            inputs[0] = INPUT {
                r#type: INPUT_MOUSE,
                Anonymous: INPUT_0 {
                    mi: MOUSE_INPUT {
                        dx: x,
                        dy: y,
                        mouseData: 0,
                        dwFlags: MOUSEEVENTF_MOVE | MOUSEEVENTF_ABSOLUTE,
                        time: 0,
                        dwExtraInfo: 0,
                    },
                },
            };

            // 2. Left Button Down
            inputs[1] = INPUT {
                r#type: INPUT_MOUSE,
                Anonymous: INPUT_0 {
                    mi: MOUSE_INPUT {
                        dx: x,
                        dy: y,
                        mouseData: 0,
                        dwFlags: MOUSEEVENTF_LEFTDOWN,
                        time: 0,
                        dwExtraInfo: 0,
                    },
                },
            };

            // 3. Left Button Up
            inputs[2] = INPUT {
                r#type: INPUT_MOUSE,
                Anonymous: INPUT_0 {
                    mi: MOUSE_INPUT {
                        dx: x,
                        dy: y,
                        mouseData: 0,
                        dwFlags: MOUSEEVENTF_LEFTUP,
                        time: 0,
                        dwExtraInfo: 0,
                    },
                },
            };

            let sent = SendInput(&inputs, size_of::<INPUT>() as i32);
            if sent == 3 {
                Ok(())
            } else {
                Err(format!("Only injected {} of 3 expected inputs.", sent))
            }
        }
    }
}


5. Data Flow

The sequence below diagrams a deterministic mouse click execution targeting a specific text label, routing through OCR fallback.

sequenceDiagram
    participant LLM as Agent Engine
    participant Core as VOXY Core Daemon (Rust)
    participant UIA as UIAutomation COM API
    participant Capture as WinRT Capture API
    participant OCR as Windows.Media.Ocr
    participant OS as SendInput API

    LLM->>Core: Request "Click 'Send Message' button"
    Core->>UIA: Query tree for "Send Message"
    alt Element Found in Tree
        UIA-->>Core: Return Automation Element Coordinates
    else Element Not Found / Blocked Custom UI Framework
        Core->>Capture: Grab screen bitmap
        Capture-->>Core: Frame buffer returned
        Core->>OCR: Execute Local OCR Analysis on frame buffer
        OCR-->>Core: Found text "Send Message" at bounding box bounds
    end
    Core->>Core: Transform and normalize coordinates to target screen space
    Core->>OS: SendInput [MOUSE_MOVE + LEFTDOWN + LEFTUP]
    OS-->>Core: Execution status verified
    Core-->>LLM: Action Complete


6. Failure Modes

HWND Z-Order Hijack: Just before VOXY executes SendInput, another application pops up in front of the target, causing the input to hit the wrong window.

UIA Tree Starvation: Querying large, complex UI trees (e.g., thousands of open tabs in a browser or IDE) blocks the tokio async core thread.

Dynamic High-DPI Switching: Dragging a window between monitors with scale factor differences ($100\%$ scaling vs $200\%$ scaling) invalidates cached coordinates.

7. Recovery Strategy

Pre-Click Verification: Immediately prior to firing SendInput, VOXY performs a micro-capture of a small $32\times32$ pixel bounding region around the target coordinates. It compares the visual signature with the cached state; if mismatch threshold $>25\%$, it pauses and re-grounds.

Background Thread Offloading: All UIA queries must run inside specialized background worker threads using tokio::task::spawn_blocking.

Dynamic Scale Factor Listening: Bind to WM_DPICHANGED messages to trigger invalidation and recalculation of all cached UI coordinates.

8. Trade-offs

Native Input Injection (SendInput) vs. UIA Patterns (Invoke): UIA InvokePattern is completely silent and works even if the window is hidden behind other apps. However, many apps do not fully support InvokePattern. Using raw coordinate projection via SendInput is universally compatible but forces the app window to visible focus.

Visual Verification Overheads: Double-checking screen pixels before executing input adds a $\approx 25\text{ms}$ latency penalty, which is acceptable because it completely prevents false-clicking.

9. Future Extension Points

Direct Hooking into DXGI Swapchains of specific applications to inject custom, fast, high-contrast overlay bounding boxes without relying on standard accessibility layers.