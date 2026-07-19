SECTION 8.2: Context Assembly & Token Budgeting Mathematics

1. Purpose

To define the mathematical algorithms that dynamically construct the LLM prompt window. We must prevent context overflow and token starvation while guaranteeing that the most critical information (System Prompt, Active UI state, core Memories) always fits within hardware limits.

2. Responsibilities

Dynamic Budgeting: Allocate token capacities based on current hardware limits ($N_{\text{max}}$).

UIA Tree Compression: Prune invisible or irrelevant UI elements to save tokens.

Memory Pruning: Evict the oldest or least relevant memories when approaching context limits.

3. Architecture

A. The Token Budget Equation

Let $T_{\text{max}}$ be the maximum context window supported by the active model/hardware (e.g., $8192$ tokens). VOXY strictly partitions this window:

$$T_{\text{max}} = T_{\text{sys}} + T_{\text{uia}} + T_{\text{mem}} + T_{\text{conv}} + T_{\text{buffer}}$$

Where:

$T_{\text{sys}}$: Immutable System Rules & MCP Schemas ($\approx 15\%$ or $1200$ tokens).

$T_{\text{uia}}$: Active Desktop State / UIA Tree ($\approx 35\%$ or $2800$ tokens).

$T_{\text{mem}}$: Retrieved Context / Knowledge Graph ($\approx 20\%$ or $1600$ tokens).

$T_{\text{conv}}$: Recent Chat History ($\approx 20\%$ or $1600$ tokens).

$T_{\text{buffer}}$: Generation Output Buffer ($\approx 10\%$ or $992$ tokens).

B. The Context Assembler Pipeline

graph TD
    A[Context Assembler Request] --> B{Determine T_max for active GPU}
    B --> C[Fetch System Prompts & MCP Tool JSON]
    B --> D[Fetch UIA Tree]
    B --> E[Fetch SQLite-Vec Memories]
    B --> F[Fetch Recent Chat Log]
    
    C --> G[Truncation / Pruning Engine]
    D --> G
    E --> G
    F --> G
    
    G -->|Strict Token Math| H[Assembled LLM Prompt]


4. Interfaces

A. Context Assembler Rust Interface (core/src/ai/context.rs)

pub struct ContextBudget {
    pub max_tokens: usize,
    pub sys_ratio: f32,
    pub uia_ratio: f32,
    pub mem_ratio: f32,
}

impl ContextAssembler {
    /// Stitches disparate states into a single prompt, truncating lower-priority data if needed.
    pub fn assemble_prompt(
        &self,
        uia_state: &UiaTree,
        memories: &[Memory],
        chat_history: &[Message],
        budget: &ContextBudget,
    ) -> String {
        // 1. Always load System (Non-negotiable)
        // 2. Compress UIA Tree to fit uia_ratio
        // 3. Filter memories by Cosine Similarity to fit mem_ratio
        // 4. Slide chat history window to fit remaining tokens
        // Return final String
    }
}


5. Data Flow (UIA Compression Algorithm)

If $T_{\text{uia}} > 2800$ tokens (e.g., user has a massive Excel spreadsheet open):

Pass 1: Remove all UIA nodes where IsOffscreen == true.

Pass 2: Remove all UIA nodes where IsEnabled == false.

Pass 3: Collapse generic Group or Pane wrappers, flattening the tree structure.

Pass 4: If still over budget, truncate nodes outside a $500\text{px}$ radius of the current mouse cursor, assuming localized user intent.

6. Failure Modes

Token Starvation for Output: The prompt consumes so much context that the LLM hits the token limit mid-generation, cutting off the MCP tool call or TTS output.

Context Dilution: Stuffing too many UI elements into the middle of the prompt causes the "Lost in the Middle" phenomenon, where the LLM ignores elements not at the very beginning or end of the context.

7. Recovery Strategy

Hard Output Buffers: The $T_{\text{buffer}}$ is a hard physical limit. The Assembler will panic and brutally truncate chat history before it ever encroaches on the output buffer.

Attention Anchoring: Place the most critical UIA elements (the active window focus) at the absolute end of the prompt, right before the Assistant generation block, to exploit the Recency Bias of transformer architectures.

8. Trade-offs

Fidelity vs. Compute Cost: Passing a massive $128\text{K}$ context window is possible with modern models, but quadratic attention calculation means Time-To-First-Token (TTFT) would spike from $150\text{ms}$ to $2+$ seconds. We aggressively compress context to $8\text{K}$ or $4\text{K}$ to preserve the sub-500ms voice latency budget.

9. Future Extension Points

KV-Cache Optimization: Pre-computing and keeping the $T_{\text{sys}}$ (System Prompt) KV-cache hot in VRAM at all times, so we only ever need to compute attention over the delta (UI and Memories) during active interactions.