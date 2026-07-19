VOXY Master Blueprint: Desktop Automation & Vision

Covers: Desktop Automation Engine, Desktop Vision

1. Purpose

To provide OpenClaw-grade deterministic desktop control without the hallucination risks, using a hybrid UIAutomation + Vision approach.

2. Responsibilities

UI Tree Extraction: Cache and query the Windows UIAutomation (UIA) tree.

Vision Grounding: Capture screen data and align OCR bounding boxes with the UIA tree.

Action Execution: Execute deterministic clicks, keystrokes, and scroll events.

3. Architecture

Addressing Phase 1 findings, we replace raw Python script generation with the Model Context Protocol (MCP) using deterministic Rust functions.

graph TD
    subgraph AI Agent Runtime
        A[Desktop Agent] -->|MCP Tool Request| B(Tool Router)
    end

    subgraph VOXY Vision / Automation Layer
        B --> C{Grounding Engine}
        
        D[UIA Caching Thread] -->|UIA Tree| C
        E[WinRT Capture] -->|Bitmap| F[Windows.Media.Ocr]
        F -->|Bounding Boxes| C
        
        C -->|Resolved Coordinates| G[Execution Engine]
    end

    G -->|SendInput / UIA Invoke| H[Windows OS]


4. Interfaces

IUIContext: get_active_window_tree() -> Json, find_element(query) -> Element.

IExecution: click(x, y), type_text(string), invoke_element(element_id).

5. Data Flow (Action Request)

LLM requests ClickElementByText(text="Submit").

Grounding Engine queries UIA cache. If found, returns coordinates/handle.

If not found in UIA, Grounding Engine triggers WinRT capture -> OCR.

If OCR finds "Submit", calculates center point.

Execution Engine calls SendInput (Mouse down, mouse up).

6. Failure Modes

UIA Unresponsive: Legacy or custom UI frameworks (e.g., heavily stylized games) return empty UIA trees.

DPI Scaling Issues: Coordinates from UIA/OCR mismatch the physical mouse coordinate system on multi-monitor setups with different scaling factors.

7. Recovery Strategy

UIA Failure: Instantly fallback to the Vision (OCR) pipeline.

DPI Mismatch: Enforce a strict coordinate normalization layer. All tools operate in a normalized [0.0 - 1.0] coordinate space; the Execution Engine maps this to physical screen pixels just before SendInput.

8. Trade-offs

Maintaining a UIA cache thread consumes background CPU, but querying UIA synchronously on every LLM request takes >200ms. Caching is worth the CPU cost for latency reduction.

9. Future Extension Points

Integration of a fast, small, local Vision Language Model (like Moondream via ONNX) specifically fine-tuned for Windows icon recognition where OCR fails (e.g., clicking a "gear" icon).