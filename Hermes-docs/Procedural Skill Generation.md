SECTION 5.3: Intelligent Procedural Memory (Dynamic Workflow Generation)

1. Purpose

To implement the highest tier of the Hermes concept—Procedural Memory—safely. Instead of generating unsafe Python scripts, VOXY observes repetitive user UI actions, mathematically extracts the workflow, and compiles it into a strict, declarative JSON State Machine (a new MCP Tool) that is deterministic and sandboxed.

2. Responsibilities

Action Observation: Hook into Windows ETW and UIA events to record user actions (with user consent).

Workflow Extraction: Identify repeating patterns in the Episodic event logs.

Skill Compilation: Translate an observed pattern into a new VOXY Model Context Protocol (MCP) tool.

3. Architecture

A. The Procedural Extraction Pipeline

sequenceDiagram
    participant User
    participant OS as Windows OS (UIA/Hooks)
    participant EpMem as Episodic Memory
    participant RefEngine as Reflection Engine (Nightly)
    participant ProcMem as Procedural Skill Registry

    User->>OS: Opens Excel, clicks "Export", saves to Desktop (Repeated 3x this week)
    OS->>EpMem: Logs structured UIA interaction sequence
    Note over RefEngine: System Idle Detected
    RefEngine->>EpMem: Scan for N-Gram repeating action sequences
    RefEngine->>RefEngine: Confidence Score > Threshold?
    RefEngine->>ProcMem: Compile sequence into deterministic JSON State Machine
    RefEngine-->>User: Next Morning: "I noticed you export reports often. I created a tool for this."


B. Mathematical Confidence Scoring

Before committing a new Workflow to Procedural memory, the Reflection Engine calculates a Confidence Score $C(w)$:


$$C(w) = \left( \sigma \times \log(F_w) \right) \times \left( 1 - \frac{V_w}{N} \right)$$


Where:

$F_w$ = Frequency of the workflow observed.

$V_w$ = Variance (number of times the user deviated or clicked 'undo' during the flow).

$N$ = Total steps in the workflow.

Rule: If $C(w) > 0.85$, VOXY proposes the new skill.

4. Interfaces

A. Generated Skill Schema (JSON State Machine)

Instead of arbitrary code, a generated skill is a strict sequence of validated VOXY primitives.

{
  "name": "auto_export_excel_report",
  "type": "procedural_workflow",
  "trigger_confidence": 0.92,
  "steps": [
    {
      "action": "uia_invoke",
      "target_id": "FileTab_Export",
      "timeout_ms": 1000
    },
    {
      "action": "uia_click",
      "target_name": "Create PDF/XPS Document",
      "fallback_ocr": "Create PDF"
    },
    {
      "action": "keyboard_type",
      "variable": "$TODAYS_DATE_FORMATTED"
    }
  ]
}


5. Data Flow

Observation: Background thread records (Window Title, Element ID, Action Type) into the SQLite Episodic table.

Analysis: The Reflection Engine uses sequential pattern mining (e.g., PrefixSpan algorithm) on the SQLite data.

Validation: The LLM is used only to name the extracted JSON workflow and define its parameters (e.g., recognizing that the typed text was a Date, and replacing it with a variable).

Registration: The JSON file is registered as a new MCP tool. The Intent Router can now call auto_export_excel_report just like a native tool.

6. Failure Modes

False Pattern Recognition: The engine incorrectly correlates unrelated actions into a broken workflow.

UI Updates: An application updates overnight, changing its UIA Element IDs, instantly breaking the generated procedural skill.

7. Recovery Strategy

Human-in-the-Loop Validation: VOXY will never execute a procedurally generated skill without asking the user the very first time it is triggered: "I generated this automation based on your habits. Should I run it?"

Self-Healing UIA Fallback: If step 2 (uia_invoke) fails, the engine automatically falls back to OCR Vision Grounding (as defined in 03_voxy_desktop_vision.md). If it succeeds via OCR, it self-heals the JSON file with the new UIA ID.

8. Trade-offs

Declarative JSON vs. Turing-Complete Scripts: By forcing procedural memory into a JSON State Machine, we lose the ability for VOXY to write complex logic (like "if this file exists, loop 10 times"). However, we gain absolute execution safety and prevent recursive terminal damage.

9. Future Extension Points

Allow users to manually "Record" a macro using their voice ("VOXY, watch me do this"), bypassing the nightly reflection requirement for immediate procedural skill compilation.