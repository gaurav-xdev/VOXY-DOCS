SECTION 5.2: Elevating Intelligence: VOXY's Hybrid Cognitive Memory Layer

1. Purpose

To replace the flawed "Vector Soup" of traditional Hermes agents with a highly intelligent, hybrid architecture. VOXY uses sqlite-vec to combine exact relational data (Knowledge Graphs) with fuzzy semantic search (Vectors) locally, governed by an asynchronous, idle-aware Reflection Engine.

2. Responsibilities

Hybrid Retrieval: Query exact relationships (SQL) and semantic meaning (Vectors) simultaneously.

Context Assembly: Dynamically construct the LLM prompt using a decay algorithm that values both recency and relevance.

Idle-State Reflection: Ensure heavy memory compression only runs when the Windows OS reports an idle user state (GetLastInputInfo).

3. Architecture

A. The VOXY Hybrid Memory Topology

Instead of just storing text, VOXY breaks down information into Entities, Relationships, and Embeddings.

graph TD
    subgraph VOXY Core Kernel
        A[Intent Router] --> B{Context Assembler}
    end

    subgraph SQLite-Vec Database [Single Local File]
        B -->|Exact Match| C[(Relational Graph)]
        B -->|Semantic Match| D[(Vector Index)]
        
        C -->|Nodes: People, Projects, Apps| E[Entity Table]
        C -->|Edges: 'Works On', 'Prefers'| F[Relationship Table]
        
        D -->|Embeddings| G[Episodic Vectors]
        D -->|Embeddings| H[Semantic Vectors]
    end

    subgraph Nightly Reflection Engine
        I[Background Learner Worker] -->|Reads| G
        I -->|Compresses & Injects| E & F & H
    end


B. Memory Decay & Retrieval Mathematics

To prevent Context Bloat, VOXY scores memory relevance using a unified formula combining Cosine Similarity, Recency Decay, and Importance:


$$S_{\text{total}} = \left( \alpha \times \frac{Q \cdot M}{\Vert{}Q\Vert{} \Vert{}M\Vert{}} \right) + \left( \beta \times e^{-\lambda t} \right) + \left( \gamma \times I_m \right)$$


Where:

$Q$ = Query Embedding, $M$ = Memory Embedding

$t$ = Time elapsed since memory creation

$\lambda$ = Decay rate constant

$I_m$ = Absolute importance score assigned by the Reflection agent (e.g., a core user preference $= 1.0$, a random conversation $= 0.1$)

4. Interfaces

A. Hybrid Memory Router Crate (core/src/memory/hybrid.rs)

use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize)]
pub struct MemoryQuery {
    pub text_query: String,
    pub embedding: Vec<f32>,
    pub required_entities: Vec<String>, // Force exact SQL match for these
}

pub trait ICognitiveMemory {
    /// Retrieves a context window optimized string containing both exact facts and fuzzy memories
    fn assemble_context(&self, query: MemoryQuery, max_tokens: usize) -> Result<String, String>;
    
    /// Commits a raw interaction to the fast Episodic buffer
    fn log_episode(&self, user_text: &str, voxy_response: &str, ui_state_hash: &str);
}


5. Data Flow

User asks: "Send the Q3 report to Sarah."

Context Assembler executes a Hybrid Query.

Relational DB finds exact Entity "Sarah" -> returns Email address.

Vector DB searches "Q3 report" -> returns Episodic memory of where the user saved a file resembling that description yesterday.

Context is injected into the LLM prompt.

6. Failure Modes

Database Locking: The background Reflection Engine locks the SQLite file for writing, blocking the real-time Voice Engine from reading context.

Entity Duplication: The LLM hallucinates two different entities for the same person (e.g., "Sarah Smith" vs "Sarah S.").

7. Recovery Strategy

WAL Mode & Read-Replicas: SQLite is configured in Write-Ahead Logging (WAL) mode. The real-time Voice thread uses a dedicated Read-Only connection, ensuring it is never blocked by the background Reflection thread.

Entity Merging: The Reflection Engine includes a deduplication phase. It calculates the Jaro-Winkler distance between entity names; if $>0.90$, it proposes a merge.

8. Trade-offs

Complexity: Managing a Hybrid Relational-Vector schema requires significantly more complex Rust boilerplate than just blindly dumping text into a Milvus instance. The trade-off is absolute deterministic accuracy for core facts (like emails and paths).

9. Future Extension Points

Implementation of Memory Time-Travel, allowing the user to say, "Revert your knowledge of my project structure to how it was last Tuesday," powered by SQLite savepoints.