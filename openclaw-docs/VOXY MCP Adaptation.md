SECTION 4.2: Hardening Agentic Control: Transitioning from OpenClaw Skills to VOXY Secure MCP Core

1. Purpose

To replace OpenClaw's insecure "Skills" architecture with a structured, local-first Model Context Protocol (MCP) framework, enforcing tight process isolation (sandboxing) and deterministic tool validation.

2. Responsibilities

Standardize Tools with MCP: Define the JSON-RPC schema layer for desktop automation tools.

Secure Tool Execution: Isolate risky tools inside Windows Sandbox (WSB) and AppContainers.

Enforce Deterministic Contracts: Ensure that the AI agent interacts with the OS only through statically compiled C++/Rust APIs, entirely preventing the runtime execution of unvalidated, AI-generated terminal scripts.

3. Architecture

A. Hardened VOXY Tool Routing Architecture

Instead of granting the LLM a shell execution command, VOXY exposes a strict registry of Model Context Protocol (MCP) tools. Any operations involving file modifications, scripting, or package management are offloaded to an isolated Windows Sandbox (WSB) instance over local TCP or named pipe channels.

graph TD
    subgraph VOXY Core [Rust System Daemon - Standard User]
        A[Agent Coordinator] -->|Evaluates Schema| B(MCP Router)
        B -->|Verify Signature & Perms| C{Is Tool Safe?}
    end

    subgraph High-Risk Sandbox [Windows Sandbox WSB]
        C -->|No: Run in Sandbox| D[Named Pipe Gateway]
        D --> E[WSB Script Executor]
        E -->|Writes/Compiles| F[Isolated File system]
    end

    subgraph Native Desktop Automation [Local Machine OS]
        C -->|Yes: Safe Native Tool| G[Rust Native UIA & Input API]
        G -->|Deterministic Action| H[Active Windows Desktop]
    end


B. Security Boundary Mapping

Safe Zone (Core Process): Performs window querying, safe desktop navigation, clipboard reading, and UIAutomation tree analysis.

Unsafe Zone (Sandbox): Installs software, executes arbitrary Python/PowerShell scripts, runs network commands, and manipulates raw files. This completely mitigates the risk of prompt-injection payloads compromising the main system.

4. Interfaces

The communication payload is standardized using the official Model Context Protocol (MCP) spec over JSON-RPC.

A. MCP Tool Definition Schema (JSON-Schema)

{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "VoxyMcpToolCall",
  "type": "object",
  "properties": {
    "method": { "type": "string", "const": "tools/call" },
    "params": {
      "type": "object",
      "properties": {
        "name": { "type": "string" },
        "arguments": { "type": "object" }
      },
      "required": ["name", "arguments"]
    }
  },
  "required": ["method", "params"]
}


B. Core Native MCP Tool: click_element

This interface replaces OpenClaw's brittle pyautogui.click(x, y) script output with an explicit, schema-enforced JSON call.

{
  "name": "desktop/click_element",
  "description": "Deterministic mouse click targeting a UI component matched via UIAutomation hierarchy.",
  "inputSchema": {
    "type": "object",
    "properties": {
      "element_id": { "type": "string", "description": "The target automation ID or localized control string." },
      "interaction_type": { "type": "string", "enum": ["left_click", "right_click", "double_click"] },
      "fallback_x": { "type": "integer" },
      "fallback_y": { "type": "integer" }
    },
    "required": ["element_id", "interaction_type"]
  }
}


5. Data Flow

This data flow shows the secure resolution of a file-writing operation, passing the payload into the WSB container rather than executing it on the local host.

sequenceDiagram
    participant LLM as Agent Engine
    participant Router as VOXY Core (Rust)
    participant Sandbox as Windows Sandbox (WSB)
    participant HostFS as Local Host Filesystem

    LLM->>Router: MCP Tool Call: "tools/call" { "name": "fs/write_file", "args": { "path": "C:\\temp\\log.txt", "content": "data" } }
    Note over Router: Intercept Tool: "fs/write_file" is marked HIGH RISK
    Router->>Sandbox: Route action via Named Pipe: "\\.\pipe\voxy_sandbox"
    Note over Sandbox: Executed in disposable VM container
    Sandbox->>Sandbox: Write "log.txt" in temporary isolated container volume
    Sandbox-->>Router: Confirm write completion (Success)
    Router-->>LLM: Return execution payload: "File safely written to virtual storage."


6. Failure Modes

WSB Cold Start Latency: Booting a new Windows Sandbox instance on tool call takes $\approx 1.5 - 2.8$ seconds, violating VOXY's sub-500ms voice loop constraint.

Sandbox IPC Disconnect: Network drops or Named Pipe pipe breakage between Core and the Sandbox VM.

Schema Drift: A plugin attempts to call an MCP tool whose interface contract has been modified in a core framework update.

7. Recovery Strategy

Warm Sandbox Pooling: VOXY maintains a single running, heavily restricted background Sandbox container in a suspended state, reducing wake latency to $< 100\text{ms}$.

IPC Watchdog: If the Named Pipe disconnects, VOXY Core restarts the sandbox orchestration worker and queues pending write buffers to prevent data loss.

Schema Validation Engine: All incoming plugin registrations are validated against strict semantic contracts on startup.

8. Trade-offs

Absolute Security vs. Developer Flexibility: Developers cannot easily write a plugin that alters local Windows registry hives directly without prompting for elevated, explicit system-administrator approval.

Performance Cost: Running a background WSB instance introduces a minimum RAM footprint overhead of $\approx 180\text{MB}$ and minor CPU context switching.

9. Future Extension Points

Distributed MCP Nodes: Allowing VOXY to route safe local MCP calls to a remote target enterprise computer running on the same Active Directory domain.

Verifiable Manifest Hashes: Using SHA-256 code signatures for verified MCP packages on a local enterprise registry.