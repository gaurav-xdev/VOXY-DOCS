SECTION 8.1: The Multi-Agent Intent Router Blueprint

1. Purpose

To design an ultra-fast, local gateway that acts as VOXY's "cerebral cortex." The Intent Router evaluates user input in real-time and dynamically routes the task to specialized sub-agents (Voice, Desktop, Memory) without introducing noticeable latency.

2. Responsibilities

Zero-Shot Classification: Instantly classify the user's intent (Chat, Action, Search, Memory Recall).

Agent Orchestration: Wake up the appropriate specialist agent and yield compute resources to it.

Grammar Constraining: Force the routing LLM to output only valid JSON enum values to guarantee parsability and speed.

3. Architecture

A. The "Gatekeeper" Model Topology

Instead of using a massive 70B parameter model to decide what to do, VOXY uses a highly quantized, extremely fast "Gatekeeper" model (e.g., a fine-tuned Microsoft Phi-3-Mini 4-bit ONNX) loaded entirely into VRAM/NPU.

graph TD
    subgraph Input Stream
        A[User Audio/Text Stream] --> B{Gatekeeper LLM (Phi-3)}
    end

    subgraph Agent Subsystems
        B -->|Intent: CHAT| C[Voice Agent]
        B -->|Intent: AUTOMATION| D[Desktop Agent]
        B -->|Intent: RETRIEVAL| E[Memory Agent]
        B -->|Intent: BROWSER| F[Browser Agent]
    end

    subgraph Execution
        C --> G[StyleTTS2 / Casual Converse]
        D --> H[UIA Tree / MCP Tool Execution]
        E --> I[SQLite-Vec Query]
        F --> J[Playwright / CDP Interface]
    end


B. Grammar-Constrained Decoding

To achieve routing decisions in $< 50\text{ms}$, we bypass standard text generation. We use Llama.cpp's grammar constraints (GBNF) to restrict the Gatekeeper's output to exactly one token representing a valid intent route.

4. Interfaces

A. Rust Router Crate (core/src/ai/router.rs)

use serde::{Deserialize, Serialize};

#[derive(Debug, Serialize, Deserialize, PartialEq)]
pub enum IntentType {
    Chat,
    DesktopAutomation,
    MemoryRecall,
    WebSearch,
}

pub struct IntentRouter {
    fast_model: GgufModel, // Kept hot in VRAM
}

impl IntentRouter {
    /// Evaluates user input against a constrained grammar to output a single enum.
    pub async fn determine_route(&self, user_prompt: &str) -> Result<IntentType, RouterError> {
        // Grammar enforces output is strictly one of the IntentType strings
        let grammar = "root ::= \"Chat\" | \"DesktopAutomation\" | \"MemoryRecall\" | \"WebSearch\"";
        self.fast_model.infer_with_grammar(user_prompt, grammar).await
    }
}


5. Data Flow

User says: "Turn off dark mode."

STT pushes text to the Intent Router.

Intent Router evaluates against system prompt: "Is this chat, desktop automation, or memory?"

GBNF constrained inference outputs: DesktopAutomation (Time-to-token: $\approx 45\text{ms}$).

The Coordinator suspends the Chat prompt and hands the text string to the Desktop Agent.

Desktop Agent triggers UIA extraction and MCP tools.

6. Failure Modes

Misclassification: The Router thinks "Remind me to call John" is a Chat intent rather than a MemoryRecall intent.

Ambiguous Prompts: The user says "Do the thing we talked about." The Router lacks the context to know if "the thing" is an automation or a search.

7. Recovery Strategy

Cross-Agent Yielding: If the Voice Agent receives a route but realizes mid-generation it lacks the capability to fulfill it, it emits an internal YIELD_TO_DESKTOP token, halting its own TTS and kicking the request over to the Desktop Agent.

Contextual Re-routing: In cases of high ambiguity, the Router asks the user for clarification before waking up heavy agent subsystems.

8. Trade-offs

Compute Overhead vs. Specialization: Running a routing model before the main agent model costs VRAM and a tiny slice of time. However, the trade-off is massively beneficial: we do not need to inject a 10,000-token UIAutomation tree into the LLM if the user is just asking, "How are you?"

9. Future Extension Points

Streaming Intent Swapping: Allowing the Intent Router to switch active agents mid-sentence as the user speaks (e.g., "Hey VOXY, how are you... actually wait, open Excel").