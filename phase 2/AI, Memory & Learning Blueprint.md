VOXY Master Blueprint: AI Runtime, Agents, Memory, & Learning

Covers: AI Runtime, Agent Runtime, Memory Engine, Learning Engine

1. Purpose

To orchestrate multi-agent routing and implement Hermes-inspired continuous learning loops using local, unified memory architecture.

2. Responsibilities

Model Routing: Determine if inference happens locally (Llama.cpp) or via cloud fallback.

Multi-Agent Orchestration: Route intents to the correct specialist agent.

4-Tier Memory Management: Handle Working, Episodic, Semantic, and Procedural memory using sqlite-vec.

Continuous Reflection: Run background processes to optimize workflows.

3. Architecture

graph TD
    subgraph Coordinator
        A[Intent Router Model (Phi-3)] 
    end

    subgraph Agents
        A -->|Casual| B[Voice Agent]
        A -->|Task| C[Desktop Agent]
        A -->|Web| D[Browser Agent]
    end

    subgraph Memory Engine [SQLite-vec DB]
        E[(Working - Context)]
        F[(Episodic - Events)]
        G[(Semantic - Facts)]
        H[(Procedural - Skills)]
    end

    subgraph Learning Engine
        I[Reflection Agent - Low Priority]
    end

    B & C & D <--> E
    I -->|Reads| F
    I -->|Writes| G & H


4. Interfaces

IMemoryStore: store_episode(context, actions), search_semantic(query_embedding, top_k).

IAgent: handle_intent(context), yield_control().

5. Data Flow (Hermes Learning Loop)

Daytime: Desktop Agent successfully completes a complex task (e.g., compiling code). Logs steps to Episodic memory.

Nighttime (Idle state detected): Reflection Agent wakes up.

Queries Episodic memory for repetitive successes/failures.

Extracts generalized steps.

Writes new MCP skill to Procedural memory.

Prunes/Deletes raw Episodic vectors to save space.

6. Failure Modes

Context Overflow: Working memory exceeds LLM token limits during long sessions.

Catastrophic Forgetting: Background reflection over-compresses data, deleting important user preferences.

Hallucinated Skills: Reflection agent generates a flawed workflow based on a one-off success.

7. Recovery Strategy

Overflow: Implement dynamic sliding-window summarization. When context hits 80%, summarize older messages.

Forgetting: Implement an immutable "Core Identity & Rules" system prompt that cannot be overwritten by the reflection loop.

Hallucinations: Procedural memory updates are flagged as unverified. Next time VOXY uses the skill, it asks for user confirmation.

8. Trade-offs

SQLite-vec over Milvus/Postgres: Sacrifices massive multi-node scale for absolute simplicity, zero configuration, and tiny RAM footprint on standard Windows machines.

9. Future Extension Points

P2P knowledge graph sharing, where VOXY instances in an enterprise securely share Procedural memory (workflows) without sharing Episodic memory (private data).