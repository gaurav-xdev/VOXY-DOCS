# 100 - Database Schema Specification

## Purpose

Define the relational database schema for VOXY's persistent data storage using SQLite.

## Scope

- Table definitions
- Index definitions
- Foreign key relationships
- Migration strategy
- Data retention policies

## Responsibilities

- Define canonical schema
- Ensure data integrity
- Support migrations
- Optimize queries
- Enforce retention

## Non-Goals

- Vector storage (Vector Database)
- Cache storage
- Log storage

## Architecture Position

Database Schema defines the persistent storage layer for all relational data.

## Schema Overview

### Tables

```sql
-- Conversations
CREATE TABLE conversations (
    id TEXT PRIMARY KEY,
    title TEXT,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    metadata TEXT -- JSON
);

-- Conversation Turns
CREATE TABLE conversation_turns (
    id TEXT PRIMARY KEY,
    conversation_id TEXT NOT NULL,
    role TEXT NOT NULL CHECK(role IN ('user', 'assistant', 'system')),
    content TEXT NOT NULL,
    timestamp DATETIME DEFAULT CURRENT_TIMESTAMP,
    metadata TEXT, -- JSON
    FOREIGN KEY (conversation_id) REFERENCES conversations(id) ON DELETE CASCADE
);

-- Episodic Events
CREATE TABLE episodic_events (
    id TEXT PRIMARY KEY,
    type TEXT NOT NULL,
    description TEXT NOT NULL,
    timestamp DATETIME DEFAULT CURRENT_TIMESTAMP,
    importance REAL DEFAULT 0.5,
    metadata TEXT, -- JSON
    embedding BLOB -- sqlite-vec
);

-- Semantic Knowledge
CREATE TABLE semantic_knowledge (
    id TEXT PRIMARY KEY,
    content TEXT NOT NULL,
    source TEXT,
    confidence REAL DEFAULT 1.0,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    embedding BLOB -- sqlite-vec
);

-- Skills / Procedural Memory
CREATE TABLE skills (
    id TEXT PRIMARY KEY,
    name TEXT NOT NULL,
    description TEXT,
    trigger_patterns TEXT, -- JSON array
    plan_template TEXT, -- JSON
    success_rate REAL DEFAULT 0.0,
    usage_count INTEGER DEFAULT 0,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    last_used DATETIME
);

-- Configuration
CREATE TABLE configuration (
    key TEXT PRIMARY KEY,
    value TEXT NOT NULL,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Audit Log
CREATE TABLE audit_log (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    timestamp DATETIME DEFAULT CURRENT_TIMESTAMP,
    subject TEXT NOT NULL,
    action TEXT NOT NULL,
    resource TEXT,
    result TEXT,
    details TEXT -- JSON
);

-- Plugin Registry
CREATE TABLE plugins (
    id TEXT PRIMARY KEY,
    name TEXT NOT NULL,
    version TEXT NOT NULL,
    author TEXT,
    path TEXT NOT NULL,
    state TEXT DEFAULT 'installed',
    permissions TEXT, -- JSON array
    installed_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Metrics
CREATE TABLE metrics (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    timestamp DATETIME DEFAULT CURRENT_TIMESTAMP,
    name TEXT NOT NULL,
    value REAL NOT NULL,
    labels TEXT -- JSON
);

-- Health Status History
CREATE TABLE health_history (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    timestamp DATETIME DEFAULT CURRENT_TIMESTAMP,
    subsystem TEXT NOT NULL,
    status TEXT NOT NULL,
    details TEXT
);
```

### Indexes

```sql
CREATE INDEX idx_turns_conversation ON conversation_turns(conversation_id, timestamp);
CREATE INDEX idx_events_timestamp ON episodic_events(timestamp);
CREATE INDEX idx_events_importance ON episodic_events(importance);
CREATE INDEX idx_knowledge_source ON semantic_knowledge(source);
CREATE INDEX idx_audit_timestamp ON audit_log(timestamp);
CREATE INDEX idx_metrics_name ON metrics(name, timestamp);
CREATE INDEX idx_health_subsystem ON health_history(subsystem, timestamp);
```

## Migration Strategy

- Migrations are versioned SQL scripts
- Applied sequentially on startup
- Rollback scripts provided
- Schema version tracked in `schema_version` table

## Data Retention

| Table | Retention |
|-------|-----------|
| conversation_turns | 90 days |
| episodic_events | 365 days |
| audit_log | 90 days |
| metrics | 30 days |
| health_history | 30 days |

## Cross References

- [101_VECTOR_DATABASE_SPEC.md](101_VECTOR_DATABASE_SPEC.md)
- [050_MEMORY_RUNTIME_SPEC.md](050_MEMORY_RUNTIME_SPEC.md)
- [112_TELEMETRY_SPEC.md](112_TELEMETRY_SPEC.md)

## References

- SQLite Schema Design
- Database Normalization
- Migration Patterns
