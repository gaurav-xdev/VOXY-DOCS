SECTION 10.2: Multi-Monitor DPI & Coordinate Math

1. Purpose

To guarantee pixel-perfect mouse clicks and bounding box generation across modern Windows setups. Mixing a 4K laptop monitor at 200% scaling with a 1080p external monitor at 100% scaling creates a chaotic virtual coordinate space that easily breaks AI automation.

2. Responsibilities

Virtual Screen Mapping: Understand the absolute topological layout of all connected monitors.

DPI Normalization: Convert coordinates from Logical (AI/UIA) space to Physical (Pixel) space.

Coordinate Translation: Map a normalized UI bounding box back to a native SendInput coordinate.

3. Architecture

A. The Windows Virtual Screen Coordinate System

Windows treats all monitors as a single massive grid called the "Virtual Screen". The primary monitor's top-left corner is ALWAYS (0, 0). Monitors to the left have negative $X$ coordinates; monitors above have negative $Y$ coordinates.

B. The Transformation Mathematics

Let a target element be found by the AI via OCR or UIA. It has a logical center coordinate $(x_l, y_l)$.
Let $S_{dpi}$ be the scaling factor of the specific monitor where the window resides (e.g., $1.5$ for $150\%$).
Let $(O_x, O_y)$ be the physical pixel offset of that monitor within the Virtual Screen.

To find the exact physical pixel $(x_p, y_p)$ to inject a mouse click:


$$x_p = (x_l \times S_{dpi}) + O_x$$

$$y_p = (y_l \times S_{dpi}) + O_y$$

To map this physical pixel to the absolute normalized coordinate space $[0, 65535]$ required by Windows SendInput (where $W_{v}$ and $H_{v}$ are the total width and height of the Virtual Screen):


$$X_{\text{input}} = \lfloor \frac{x_p \times 65535}{W_v} \rfloor$$

$$Y_{\text{input}} = \lfloor \frac{y_p \times 65535}{H_v} \rfloor$$

4. Interfaces

A. Rust DPI Normalization Engine (core/src/vision/dpi.rs)

use windows::Win32::UI::WindowsAndMessaging::{GetSystemMetrics, SM_CXVIRTUALSCREEN, SM_CYVIRTUALSCREEN, SM_XVIRTUALSCREEN, SM_YVIRTUALSCREEN};
use windows::Win32::UI::HiDpi::GetDpiForWindow;
use windows::Win32::Foundation::HWND;

pub struct DpiEngine;

impl DpiEngine {
    /// Translates logical window coordinates to SendInput normalized space [0-65535]
    pub fn translate_to_absolute_input(hwnd: HWND, logical_x: i32, logical_y: i32) -> (i32, i32) {
        unsafe {
            // 1. Get DPI Scale for the specific window's monitor
            let dpi = GetDpiForWindow(hwnd) as f32;
            let scale_factor = dpi / 96.0;

            // 2. Convert to Physical Pixels
            let physical_x = (logical_x as f32 * scale_factor) as i32;
            let physical_y = (logical_y as f32 * scale_factor) as i32;

            // 3. Get Virtual Screen Dimensions
            let v_width = GetSystemMetrics(SM_CXVIRTUALSCREEN);
            let v_height = GetSystemMetrics(SM_CYVIRTUALSCREEN);
            let v_left = GetSystemMetrics(SM_XVIRTUALSCREEN);
            let v_top = GetSystemMetrics(SM_YVIRTUALSCREEN);

            // 4. Map to absolute [0, 65535] space
            let mapped_x = ((physical_x - v_left) * 65535) / v_width;
            let mapped_y = ((physical_y - v_top) * 65535) / v_height;

            (mapped_x, mapped_y)
        }
    }
}


5. Data Flow

Desktop Agent outputs an MCP tool call to click "Settings".

UIA Tree returns logical bounds of the "Settings" button.

DpiEngine evaluates which monitor the target application currently resides on.

Mathematical translation calculates the required $65535$ scale mapping.

OS SendInput API physically moves the mouse to the precise center of the button, regardless of scaling.

6. Failure Modes

Mid-Execution Topology Changes: The user unplugs their laptop from a dock while VOXY is in the middle of executing a 5-step automation, instantly changing the Virtual Screen size and invalidating all cached coordinates.

Per-Monitor V2 Awareness: Legacy apps that are not DPI-aware will lie about their logical coordinates, causing VOXY to click in empty space.

7. Recovery Strategy

WM_DISPLAYCHANGE Listener: VOXY's kernel listens for WM_DISPLAYCHANGE. If triggered, all active automation agents instantly pause, flush their UIA/OCR caches, and re-ground the screen.

Micro-Verification: Before clicking, VOXY uses WinRT to capture a $32\times32$ pixel box around the target $(x_p, y_p)$ to verify color/edge density matches the expected UI element.

8. Trade-offs

Mathematical Complexity vs. OpenClaw Naivety: OpenClaw routinely fails when moving from a Mac to a Windows machine because it hardcodes physical pixels. VOXY's DPI math introduces minimal CPU overhead but guarantees multi-hardware execution success.

9. Future Extension Points

Handling dynamic refresh rates and VRR (Variable Refresh Rate) setups where DWM composition tearlines might temporarily distort OCR reads during high-framerate gaming.