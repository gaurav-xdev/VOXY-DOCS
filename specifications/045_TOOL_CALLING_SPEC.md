# 045 - Tool Calling Specification

## Purpose

Define the Tool Calling subsystem that manages the discovery, invocation, and result handling of external tools and functions available to the cognitive engine.

## Scope

- Tool schema management
- Tool discovery and registration
- Function invocation
- Result serialization
- Error handling and retry
- Tool permission enforcement

## Responsibilities

- Register and manage tool schemas
- Validate tool invocations
- Execute tool calls
- Serialize results for LLM consumption
- Handle tool errors
- Enforce tool permissions

## Non-Goals

- Tool implementation (individual tools)
- LLM function calling protocol (Model Router)
- Tool UI (separate component)

## Architecture Position

Tool Calling sits between Executor and individual tool implementations. It is the bridge between LLM function calls and actual execution.

## Inputs

- Tool call requests from Executor
- Tool schemas from plugins and built-ins
- Tool implementations
- Permission grants

## Outputs

- Tool execution results
- Schema catalogs
- Error responses

## Interfaces

```rust
pub struct ToolCalling {
    pub fn register_tool(schema: ToolSchema, implementation: Box<dyn Tool>) -> Result<(), ToolError>;
    pub fn list_tools() -> Vec<ToolSchema>;
    pub fn invoke(call: ToolCall) -> Result<ToolResult, ToolError>;
    pub fn validate_call(call: &ToolCall) -> Result<(), ValidationError>;
}
```

## Data Models

### ToolSchema

| Field | Type | Description |
|-------|------|-------------|
| name | String | Tool identifier |
| description | String | Tool description |
| parameters | JSONSchema | Parameter schema |
| returns | JSONSchema | Return schema |
| required_permissions | Vec<String> | Required capabilities |

### ToolCall

| Field | Type | Description |
|-------|------|-------------|
| id | String | Call identifier |
| name | String | Tool name |
| arguments | Map | Named arguments |

### ToolResult

| Field | Type | Description |
|-------|------|-------------|
| call_id | String | Call identifier |
| success | bool | Execution success |
| result | Option<String> | JSON result |
| error | Option<String> | Error message |
| duration_ms | u64 | Execution time |

## State Management

- Tool registry is maintained in-memory
- Schemas are loaded at startup
- Tool state is ephemeral

## Threading Model

Tool invocations run on Tokio runtime. Long-running tools may use blocking pool.

## IPC Requirements

- Plugin tools may execute in sandbox via IPC
- Built-in tools are in-process

## Event Definitions

| Event | Payload | Description |
|-------|---------|-------------|
| `tool.registered` | ToolSchema | Tool registered |
| `tool.invoked` | ToolCall | Tool called |
| `tool.completed` | ToolResult | Tool finished |
| `tool.failed` | (call_id, error) | Tool failed |

## Error Handling

| Code | Description | Recovery |
|------|-------------|----------|
| `TC001` | Tool not found | Return error to LLM |
| `TC002` | Invalid arguments | Return validation error |
| `TC003` | Permission denied | Return auth error |
| `TC004` | Tool timeout | Cancel, return timeout |
| `TC005` | Tool panic | Isolate, return error |

## Retry Strategy

- Tool invocation: Configurable per tool
- Default: 1 retry

## Recovery Strategy

- Tool failure: Return error to LLM
- Timeout: Cancel and return timeout
- Panic: Isolate tool, restart if possible

## Security Requirements

- Tools require explicit capability grants
- Plugin tools run in sandbox
- Arguments are validated against schema
- No arbitrary code execution

## Configuration

```json
{
  "tool_calling": {
    "max_concurrent_tools": 10,
    "default_timeout_seconds": 30,
    "enable_schema_validation": true,
    "strict_permissions": true,
    "tool_result_max_length": 10000
  }
}
```

## Performance Targets

| Metric | Target |
|--------|--------|
| Tool dispatch | < 5ms |
| Schema validation | < 1ms |
| Result serialization | < 1ms |

## Resource Limits

| Resource | Limit |
|----------|-------|
| Registered tools | 500 |
| Concurrent invocations | 10 |
| Result size | 10KB |

## Benchmarks

- `bench_dispatch`: Tool dispatch time
- `bench_validation`: Schema validation speed
- `bench_concurrent`: Concurrent tool load

## Testing Requirements

- Schema validation tests
- Permission enforcement tests
- Timeout handling tests
- Error propagation tests

## Logging Requirements

- Tool invocations at DEBUG level
- Errors at ERROR level
- Schema violations at WARN level

## Telemetry

- Tool invocation frequency
- Tool success rate
- Average execution time
- Permission denial rate

## Cross References

- [042_PLANNER_SPEC.md](042_PLANNER_SPEC.md)
- [043_EXECUTOR_SPEC.md](043_EXECUTOR_SPEC.md)
- [080_PLUGIN_RUNTIME_SPEC.md](080_PLUGIN_RUNTIME_SPEC.md)
- [082_PERMISSION_MODEL_SPEC.md](082_PERMISSION_MODEL_SPEC.md)

## References

- OpenAI Function Calling
- Anthropic Tool Use
- LangChain Tools
- JSON Schema
