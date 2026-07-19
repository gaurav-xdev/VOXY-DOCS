SECTIONS 4, 5, 6, 8, 9: Automation, Memory, and Agents

1. Desktop Automation: OpenClaw Post-Mortem (Section 4)

Analysis: OpenClaw relies on a monolithic LLM prompt to generate Python scripts using pyautogui based on screenshots.

Weaknesses: Brittle to resolution changes. High latency. Unsafe execution.

The VOXY Redesign: Implement the Model Context Protocol (MCP). VOXY will not write scripts. Instead, VOXY agents call deterministic, pre-compiled C++/Rust functions (e.g., ClickElementByUIA(id="submit_btn")).

2. Multi-Agent System (Section 9)

Note on "Agency Agent": While standard actor-model frameworks exist, VOXY requires a deeply integrated, low-latency coordinator.

Architecture:

Coordinator Agent: A fast routing model (e.g., Phi-3). Determines if the user intent is "Chat", "Action", or "Search".

Voice Agent: Handles casual conversation and empathy.

Desktop Agent: Given a goal, retrieves the UI tree, grounds it, and emits MCP tool calls.

Observer/Reflection Agent: Runs in the background (low CPU priority). Watches successful tasks and writes them to Memory.

3. The Hermes Learning System (Section 5 & 6)

VOXY will implement a continuous learning loop mirroring human memory:

Working Memory: In-memory context of the current conversation (Context window).

Episodic Memory (Events): "What did I do yesterday?"

Semantic Memory (Knowledge/Preferences): "The user prefers dark mode."

Procedural Memory (Workflows): "How do I compile this project?"

Storage Architecture (Section 6):

Avoid: Heavy standalone DBs like Postgres/Milvus.

Recommendation: SQLite with sqlite-vec extension. It allows relational data and vector embeddings (for semantic search) in a single, local, zero-config file.

Compression/Forgetting: An idle background task runs nightly. It summarizes old Episodic memories, extracts Semantic facts, and deletes the raw vectors to preserve local disk space.