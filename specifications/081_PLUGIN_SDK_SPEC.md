# 081 - Plugin SDK Specification

## Purpose

Define the Plugin SDK that provides the API surface, development tools, and integration patterns for third-party developers to create VOXY plugins.

## Scope

- Plugin API definitions
- Event subscription API
- Tool registration API
- UI extension points
- Lifecycle callbacks
- Type definitions and schemas

## Responsibilities

- Define plugin API contracts
- Provide event access
- Enable tool registration
- Support UI extensions
- Document integration patterns
- Provide development utilities

## Non-Goals

- Plugin runtime management (Plugin Runtime)
- Sandbox implementation (Sandbox)
- Plugin store/marketplace

## Architecture Position

Plugin SDK is the public API that plugins consume. It communicates with Plugin Runtime via IPC.

## Plugin API Surface

### Core API

```typescript
// Conceptual interface - specification only
interface VoxyPlugin {
  id: string;
  name: string;
  version: string;

  onLoad(context: PluginContext): void | Promise<void>;
  onUnload(): void | Promise<void>;
  onEvent(event: VoxyEvent): void | Promise<void>;
}

interface PluginContext {
  events: EventBus;
  tools: ToolRegistry;
  config: ConfigManager;
  logger: Logger;
  storage: Storage;
}

interface EventBus {
  subscribe(topic: string, handler: EventHandler): Subscription;
  publish(topic: string, payload: any): void;
}

interface ToolRegistry {
  registerTool(schema: ToolSchema, handler: ToolHandler): void;
}
```

## Data Models

### PluginManifest

| Field | Type | Description |
|-------|------|-------------|
| id | String | Unique plugin ID |
| name | String | Display name |
| version | SemVer | Plugin version |
| author | String | Author name |
| description | String | Description |
| permissions | Vec<String> | Required permissions |
| capabilities | Vec<String> | Provided capabilities |
| entry_point | String | Main module path |
| min_voxy_version | SemVer | Minimum VOXY version |

### ToolSchema

| Field | Type | Description |
|-------|------|-------------|
| name | String | Tool name |
| description | String | Tool description |
| parameters | JSONSchema | Parameter schema |
| returns | JSONSchema | Return schema |

## Security Requirements

- Plugins declare required permissions
- Permissions enforced by Sandbox
- No direct OS access
- All I/O via SDK APIs

## Configuration

```json
{
  "plugin_sdk": {
    "api_version": "1.0.0",
    "supported_languages": ["javascript", "typescript", "python"],
    "max_event_handlers": 100,
    "max_tools_per_plugin": 20,
    "storage_quota_mb": 100
  }
}
```

## Cross References

- [080_PLUGIN_RUNTIME_SPEC.md](080_PLUGIN_RUNTIME_SPEC.md)
- [082_PERMISSION_MODEL_SPEC.md](082_PERMISSION_MODEL_SPEC.md)
- [083_SANDBOX_SPEC.md](083_SANDBOX_SPEC.md)

## References

- VS Code Extension API
- Chrome Extension API
- Node.js Addon API
