SECTION 10.3: Vision-Language Model (VLM) Fallback Architecture

1. Purpose

To provide VOXY with deep semantic screen understanding when both UIA and OCR fail. For example, if a user asks to "Click the gear icon" or "Save the image of the golden retriever", neither the UI tree nor text OCR can resolve these purely visual concepts. A local VLM bridges this gap.

2. Responsibilities

Visual Target Identification: Process raw screen pixels and a text description to return precise bounding boxes.

Contextual Screenshotting: Cropping the capture to the active window to save VRAM and processing time.

Graceful Latency Management: Pausing the conversational Voice Agent to inform the user that a complex visual task is underway.

3. Architecture

A. The Tiered Vision Fallback Graph

The VLM is treated as the "last resort" engine due to its computational expense.

graph TD
    A[Agent Requests Target: 'Gear Icon'] --> B{Layer 1: UIA Tree}
    B -->|Miss| C{Layer 2: Local OCR}
    C -->|Miss| D[Layer 3: VLM Fallback]
    
    subgraph VLM Execution Subsystem
        D --> E[Crop Screen to Active Window]
        E --> F[Load Small VLM into NPU/VRAM]
        F -->|Input: Image + 'Find Gear Icon'| G{Moondream2 ONNX}
        G -->|Output: [x1, y1, x2, y2]| H[Coordinate Normalizer]
    end
    
    H --> I[Execute Action]


4. Interfaces

A. VLM Execution Crate (core/src/vision/vlm.rs)

pub struct VlmEngine {
    model_path: String,
    execution_provider: String, // e.g., "DirectML" or "CPU"
}

impl VlmEngine {
    /// Queries the VLM to find the bounding box of a semantic description.
    /// This is an expensive, blocking async operation.
    pub async fn locate_visual_element(
        &self, 
        image_bytes: &[u8], 
        description: &str
    ) -> Result<Vec<f32>, String> {
        // Formulate prompt specifically for grounding/bounding boxes
        let prompt = format!("Find the exact bounding box for: {}. Output strictly in [ymin, xmin, ymax, xmax] format.", description);
        
        // Pseudo-code for ONNX Runtime invocation
        // let output = onnx_run(self.model_path, image_bytes, prompt).await?;
        // Ok(parse_bounding_box(output))
        
        unimplemented!()
    }
}


5. Data Flow

User says: "Like the second video in my feed."

Intent Router assigns task to Desktop Agent.

Desktop Agent searches UIA for "Like" (Fails: Web custom canvas).

Desktop Agent searches OCR for "Like" (Fails: It's a thumbs-up icon, not text).

Desktop Agent invokes VLM Fallback Tool: locate_visual_element("thumbs up icon").

Voice Agent is notified and says: "Scanning the screen for that..." (buying time).

Moondream2 processes the frame ($\approx 1.2\text{s}$ on GPU, $\approx 3.5\text{s}$ on CPU).

VLM returns normalized coordinates [0.45, 0.30, 0.48, 0.33].

DPI Engine maps to absolute pixels; execution completes.

6. Failure Modes

Hardware Starvation: Loading a 2B parameter VLM into VRAM while the main Llama-3 8B model is also loaded causes an Out-Of-Memory (OOM) crash on 8GB GPUs.

Hallucinated Coordinates: The VLM successfully identifies that a gear icon exists, but outputs coordinates that are slightly off, causing VOXY to click the background.

7. Recovery Strategy

Model Swapping (Memory Management): Before loading the VLM, the Execution Router temporarily offloads the Llama-3 KV-Cache to system RAM, clearing space. Once the vision task is complete, it swaps the cache back into VRAM.

Cloud Vision Fallback: If the local hardware cannot support the VLM without crashing (e.g., Intel integrated graphics), the image is routed to a secure cloud VLM (like GPT-4o-mini) only if the user has explicitly opted into cloud vision processing.

8. Trade-offs

Speed vs. Capability: A purely UIA/OCR system is lightning fast ($< 100\text{ms}$) but rigid. Adding a VLM makes VOXY truly "see" like a human, but shatters the $500\text{ms}$ latency budget, requiring the UX to dynamically adjust (e.g., using audio cues to signal processing).

9. Future Extension Points

Fine-tuning a micro-model (e.g., Florence-2-base, $< 0.3\text{B}$ params) specifically on Windows 11 icon sets to run continuously in the background using $< 200\text{MB}$ of VRAM.