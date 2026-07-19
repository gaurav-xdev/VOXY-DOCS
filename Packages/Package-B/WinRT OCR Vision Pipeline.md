SECTION 10.1: WinRT Zero-Copy OCR & Vision Pipeline

1. Purpose

To provide VOXY with a sub-100ms visual understanding of the screen when UIAutomation (UIA) fails or is unavailable. This pipeline bypasses slow, CPU-heavy capture methods (like BitBlt) in favor of zero-copy GPU capture and highly optimized local Windows native OCR.

2. Responsibilities

Screen Capture: Utilize Windows.Graphics.Capture to grab frames directly from the Desktop Window Manager (DWM) composition swapchain.

Text Extraction: Process the captured frame through Windows.Media.Ocr.OcrEngine entirely locally.

Bounding Box Mapping: Map recognized text lines and words back to relative screen coordinates for the Execution Engine to target.

3. Architecture

A. The Zero-Copy Capture Pipeline

By keeping the captured frame in GPU memory (Direct3D 11) as long as possible, we avoid expensive CPU-RAM copies that cause latency spikes.

graph TD
    subgraph Desktop Window Manager (OS)
        A[DWM Composition Surface]
    end

    subgraph VOXY Vision Worker (Rust)
        B[GraphicsCaptureItem] -->|Frame Arrives| C[Direct3D11 Texture2D]
        C -->|Convert| D[SoftwareBitmap]
        D -->|Pass to OCR Engine| E[Windows.Media.Ocr.OcrEngine]
    end

    subgraph OCR Results
        E -->|Recognized Text| F[OcrResult]
        F --> G[List of OcrLine / OcrWord]
        G -->|Bounding Rectangles| H[Grounding Engine]
    end
    
    A -->|WinRT Capture API| B


4. Interfaces

A. Vision Pipeline Rust Interface (core/src/vision/capture.rs)

use windows::Graphics::Capture::GraphicsCaptureItem;
use windows::Graphics::Imaging::SoftwareBitmap;
use windows::Media::Ocr::{OcrEngine, OcrResult};
use windows::Win32::Foundation::RECT;

pub struct VisionPipeline {
    ocr_engine: OcrEngine,
}

impl VisionPipeline {
    /// Initializes the OCR Engine for the user's current system language
    pub fn new() -> Result<Self, String> {
        // Enforce local execution only
        let engine = OcrEngine::TryCreateFromUserProfileLanguages()?;
        Ok(Self { ocr_engine: engine })
    }

    /// Captures the current screen state and extracts text bounding boxes
    pub async fn extract_screen_text(&self, target: &GraphicsCaptureItem) -> Result<Vec<(String, RECT)>, String> {
        let bitmap: SoftwareBitmap = self.capture_frame(target).await?;
        let result: OcrResult = self.ocr_engine.RecognizeAsync(&bitmap)?.await?;
        
        let mut text_elements = Vec::new();
        for line in result.Lines()? {
            for word in line.Words()? {
                text_elements.push((word.Text()?.to_string(), word.BoundingRect()?));
            }
        }
        Ok(text_elements)
    }
}


5. Data Flow

Fallback Triggered: The Desktop Agent fails to find the "Save" button via UIA.

Capture: The Vision Pipeline requests a frame of the currently active display monitor via GraphicsCaptureItem.

OCR: The GPU-backed SoftwareBitmap is processed by the Windows native OCR engine ($\approx 40\text{ms}$).

Resolution: The Grounding Engine searches the resulting Vec<(String, RECT)> for a fuzzy match to the word "Save".

Execution: The exact bounding box RECT is returned to the Input Driver for a mouse click.

6. Failure Modes

Anti-Cheat Blocks: Some video games hook into the DWM and block Windows.Graphics.Capture to prevent aimbots.

Low Contrast Text: Gray text on a dark gray background often fails standard OCR thresholding.

Language Mismatch: The target application is displaying text in French, but the OcrEngine is initialized for en-US.

7. Recovery Strategy

Capture Fallback: If GraphicsCaptureItem throws an access denied error, VOXY falls back to the legacy GDI BitBlt capture API.

Pre-Processing (Binarization): If OCR returns zero results, VOXY runs a rapid compute shader to convert the image to high-contrast black-and-white (binarization) and attempts OCR a second time.

8. Trade-offs

WinRT API vs. Tesseract: We use Windows native OCR because it requires zero additional bundled models (saving 100MB+ in distribution size) and is highly hardware-optimized by Microsoft. However, it requires Windows 10+ and limits our cross-platform aspirations.

9. Future Extension Points

Injecting localized OCR models on-the-fly when the AI Agent detects that the user is interacting with an application in a foreign language.