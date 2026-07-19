# 110 - Configuration Schema Specification

## Purpose

Define the complete JSON schema for VOXY configuration files, ensuring validation, type safety, and documentation of all configurable parameters.

## Scope

- Configuration file schema
- Validation rules
- Default values
- Environment variable mapping
- Configuration hierarchy

## Responsibilities

- Define canonical schema
- Validate configuration
- Document all options
- Enforce constraints
- Support migrations

## Non-Goals

- Configuration UI
- Remote configuration
- Configuration versioning

## Schema

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "kernel": {
      "type": "object",
      "properties": {
        "thread_pool_size": { "type": "integer", "minimum": 1, "default": 16 },
        "memory_quota_mb": { "type": "integer", "minimum": 256, "default": 2048 },
        "init_timeout_seconds": { "type": "integer", "minimum": 1, "default": 30 },
        "shutdown_timeout_seconds": { "type": "integer", "minimum": 1, "default": 30 },
        "enable_telemetry": { "type": "boolean", "default": true },
        "log_level": { "type": "string", "enum": ["TRACE", "DEBUG", "INFO", "WARN", "ERROR"], "default": "INFO" },
        "crash_reporting": { "type": "boolean", "default": true }
      }
    },
    "audio_pipeline": {
      "type": "object",
      "properties": {
        "sample_rate": { "type": "integer", "enum": [8000, 16000, 44100, 48000], "default": 16000 },
        "channels": { "type": "integer", "minimum": 1, "maximum": 2, "default": 1 },
        "format": { "type": "string", "enum": ["f32", "i16", "i32"], "default": "f32" },
        "buffer_size_ms": { "type": "integer", "minimum": 10, "maximum": 1000, "default": 100 },
        "latency_budget_ms": { "type": "integer", "minimum": 100, "maximum": 5000, "default": 500 },
        "enable_barge_in": { "type": "boolean", "default": true },
        "stt_timeout_seconds": { "type": "integer", "minimum": 1, "default": 10 },
        "tts_timeout_seconds": { "type": "integer", "minimum": 1, "default": 30 }
      }
    },
    "model_router": {
      "type": "object",
      "properties": {
        "primary_provider": { "type": "string", "enum": ["Local", "OpenAI", "Anthropic", "Azure", "Ollama"], "default": "Local" },
        "fallback_chain": {
          "type": "array",
          "items": { "type": "string" },
          "default": ["Azure", "OpenAI", "Anthropic"]
        },
        "default_model": { "type": "string", "default": "phi-silica" },
        "max_latency_ms": { "type": "integer", "minimum": 1000, "default": 5000 },
        "enable_cost_tracking": { "type": "boolean", "default": true },
        "privacy_mode": { "type": "string", "enum": ["local_only", "local_first", "balanced", "cloud_first"], "default": "local_first" },
        "token_budget_daily": { "type": "integer", "minimum": 0, "default": 100000 }
      }
    },
    "memory_runtime": {
      "type": "object",
      "properties": {
        "database_path": { "type": "string", "default": "%LOCALAPPDATA%/VOXY/memory.db" },
        "max_size_mb": { "type": "integer", "minimum": 100, "default": 1024 },
        "consolidation_interval_hours": { "type": "integer", "minimum": 1, "default": 24 },
        "embedding_model": { "type": "string", "default": "local" },
        "embedding_dimensions": { "type": "integer", "enum": [384, 768, 1536], "default": 384 },
        "enable_encryption": { "type": "boolean", "default": true },
        "backup_interval_days": { "type": "integer", "minimum": 1, "default": 7 }
      }
    },
    "automation_runtime": {
      "type": "object",
      "properties": {
        "action_timeout_seconds": { "type": "integer", "minimum": 1, "default": 10 },
        "element_find_retries": { "type": "integer", "minimum": 0, "default": 3 },
        "screen_capture_interval_ms": { "type": "integer", "minimum": 100, "default": 500 },
        "element_cache_ttl_ms": { "type": "integer", "minimum": 100, "default": 1000 },
        "max_concurrent_actions": { "type": "integer", "minimum": 1, "default": 1 },
        "require_confirmation_for": {
          "type": "array",
          "items": { "type": "string" },
          "default": ["file_delete", "install", "run_as_admin"]
        }
      }
    },
    "security_model": {
      "type": "object",
      "properties": {
        "lockout_after_failures": { "type": "integer", "minimum": 1, "default": 5 },
        "lockout_duration_minutes": { "type": "integer", "minimum": 1, "default": 30 },
        "session_ttl_minutes": { "type": "integer", "minimum": 1, "default": 60 },
        "require_mfa": { "type": "boolean", "default": false },
        "audit_all_actions": { "type": "boolean", "default": true }
      }
    },
    "plugin_runtime": {
      "type": "object",
      "properties": {
        "plugin_directory": { "type": "string", "default": "%LOCALAPPDATA%/VOXY/plugins/" },
        "max_plugins": { "type": "integer", "minimum": 1, "default": 50 },
        "sandbox_enabled": { "type": "boolean", "default": true },
        "auto_restart": { "type": "boolean", "default": true },
        "max_restart_count": { "type": "integer", "minimum": 0, "default": 3 },
        "code_signing_required": { "type": "boolean", "default": true }
      }
    },
    "logging": {
      "type": "object",
      "properties": {
        "level": { "type": "string", "enum": ["TRACE", "DEBUG", "INFO", "WARN", "ERROR"], "default": "INFO" },
        "format": { "type": "string", "enum": ["json", "pretty"], "default": "json" },
        "output_path": { "type": "string", "default": "%LOCALAPPDATA%/VOXY/logs/" },
        "rotation": {
          "type": "object",
          "properties": {
            "max_size_mb": { "type": "integer", "default": 100 },
            "max_files": { "type": "integer", "default": 10 },
            "max_age_days": { "type": "integer", "default": 30 }
          }
        },
        "console": { "type": "boolean", "default": false },
        "windows_event_log": { "type": "boolean", "default": true }
      }
    },
    "metrics": {
      "type": "object",
      "properties": {
        "enabled": { "type": "boolean", "default": true },
        "collection_interval_seconds": { "type": "integer", "minimum": 1, "default": 15 },
        "export_interval_seconds": { "type": "integer", "minimum": 1, "default": 60 },
        "max_cardinality": { "type": "integer", "minimum": 100, "default": 10000 },
        "backends": {
          "type": "array",
          "items": { "type": "string", "enum": ["prometheus", "otlp"] },
          "default": ["prometheus"]
        },
        "prometheus_port": { "type": "integer", "minimum": 1024, "maximum": 65535, "default": 9090 }
      }
    }
  }
}
```

## Cross References

- [014_CONFIGURATION_SPEC.md](014_CONFIGURATION_SPEC.md)
- [010_KERNEL_SPEC.md](010_KERNEL_SPEC.md)

## References

- JSON Schema Draft 2020-12
- Configuration Management Patterns
