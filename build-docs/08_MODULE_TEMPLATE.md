# VOXY — Module Template

| Field | Value |
|-------|-------|
| **Version** | 1.0.0 |
| **Status** | Production-Ready |
| **Last Updated** | 2026-07-17 |
| **Author** | VOXY Engineering Team |
| **Classification** | Internal — Scaffolding Foundation |

---

## Purpose

This document provides the standard template for creating new modules in the VOXY codebase. Every new module must follow this structure to ensure consistency, testability, and maintainability.

---

## Scope

Covers:
- Module directory structure
- Cargo.toml template
- lib.rs template
- error.rs template
- config.rs template
- engine.rs template
- README.md template
- Verification commands

---

## Audience

- Engineers creating new modules
- AI Coding Agents generating module scaffolding
- Tech Leads reviewing module structure

---

## Module Directory Structure

```
src/{module_name}/
├── Cargo.toml              # Module manifest
├── README.md               # Module documentation
├── src/
│   ├── lib.rs              # Public API exports
│   ├── error.rs            # Error types
│   ├── config.rs           # Configuration structs
│   ├── types.rs            # Core domain types
│   ├── engine.rs           # Main engine/implementation
│   ├── handlers.rs         # Event handlers
│   └── utils.rs            # Internal utilities
├── tests/
│   ├── unit_tests.rs       # Unit tests
│   ├── integration_tests.rs # Integration tests
│   └── fixtures/           # Test data
└── benches/
    └── benchmark.rs         # Criterion benchmarks
```

---

## Cargo.toml Template

```toml
[package]
name = "voxy-{module_name}"
version.workspace = true
edition.workspace = true
authors.workspace = true
license.workspace = true
repository.workspace = true

[dependencies]
# Workspace dependencies
tokio = { workspace = true }
serde = { workspace = true }
thiserror = { workspace = true }
anyhow = { workspace = true }
tracing = { workspace = true }

# Module-specific dependencies
# (Add only what this module needs)

[dev-dependencies]
mockall = { workspace = true }
proptest = { workspace = true }
criterion = { workspace = true }
tokio-test = "0.4"

[[bench]]
name = "benchmark"
harness = false
```

---

## lib.rs Template

```rust
//! # VOXY {Module Name}
//!
//! {One-line description of module purpose.}
//!
//! ## Overview
//!
//! {Detailed description of what this module does, its responsibilities,
//! and how it fits into the VOXY architecture.}
//!
//! ## Architecture
//!
//! ```text
//! {ASCII diagram of internal component structure}
//! ```
//!
//! ## Usage
//!
//! ```rust,no_run
//! use voxy_{module_name}::{Engine, Config};
//!
//! let config = Config::default();
//! let engine = Engine::new(config).await?;
//! engine.start().await?;
//! ```
//!
//! ## Module ID
//!
//! **MOD-{ID}**

pub mod config;
pub mod error;
pub mod types;

pub use config::Config;
pub use error::{Error, Result};
pub use types::*;
pub use engine::Engine;

mod engine;
mod handlers;
mod utils;
```

---

## error.rs Template

```rust
//! Error types for MOD-{ID}

use thiserror::Error;

/// Errors that can occur in the {module_name} module.
#[derive(Error, Debug)]
pub enum Error {
    /// Configuration is invalid or missing required fields.
    #[error("invalid configuration: {0}")]
    InvalidConfig(String),

    /// The requested resource was not found.
    #[error("not found: {0}")]
    NotFound(String),

    /// An operation timed out.
    #[error("operation timed out after {0:?}")]
    Timeout(std::time::Duration),

    /// An internal error occurred.
    #[error("internal error: {0}")]
    Internal(String),

    /// A wrapped I/O error.
    #[error("I/O error: {0}")]
    Io(#[from] std::io::Error),
}

/// Result type alias for this module.
pub type Result<T> = std::result::Result<T, Error>;
```

---

## config.rs Template

```rust
//! Configuration for MOD-{ID}

use serde::{Deserialize, Serialize};

/// Configuration for the {module_name} module.
#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct Config {
    /// Enable or disable this module.
    #[serde(default = "default_enabled")]
    pub enabled: bool,

    /// Timeout for operations in seconds.
    #[serde(default = "default_timeout_secs")]
    pub timeout_secs: u64,
}

impl Default for Config {
    fn default() -> Self {
        Self {
            enabled: default_enabled(),
            timeout_secs: default_timeout_secs(),
        }
    }
}

fn default_enabled() -> bool {
    true
}

fn default_timeout_secs() -> u64 {
    30
}
```

---

## engine.rs Template

```rust
//! Core engine implementation for MOD-{ID}

use tracing::{info, error, debug};
use crate::{Config, Error, Result};

/// The main engine for the {module_name} module.
pub struct Engine {
    config: Config,
    // Add internal state fields
}

impl Engine {
    /// Create a new engine instance with the given configuration.
    ///
    /// # Errors
    ///
    /// Returns `Error::InvalidConfig` if the configuration is invalid.
    pub fn new(config: Config) -> Result<Self> {
        // Validate configuration
        if !config.enabled {
            return Err(Error::InvalidConfig("module is disabled".to_string()));
        }

        Ok(Self {
            config,
        })
    }

    /// Start the engine.
    ///
    /// # Errors
    ///
    /// Returns `Error::Internal` if the engine fails to start.
    pub async fn start(&self) -> Result<()> {
        info!("Starting {module_name} engine");
        // Implementation
        Ok(())
    }

    /// Stop the engine gracefully.
    pub async fn stop(&self) -> Result<()> {
        info!("Stopping {module_name} engine");
        // Implementation
        Ok(())
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn engine_creates_with_valid_config() {
        let config = Config::default();
        let engine = Engine::new(config);
        assert!(engine.is_ok());
    }

    #[test]
    fn engine_fails_with_disabled_config() {
        let mut config = Config::default();
        config.enabled = false;
        let engine = Engine::new(config);
        assert!(matches!(engine, Err(Error::InvalidConfig(_))));
    }
}
```

---

## README.md Template

```markdown
# MOD-{ID}: {Module Name}

## Purpose

{One-paragraph description of module responsibility.}

## Dependencies

| Module | Purpose |
|--------|---------|
| MOD-{X} | {Why this module is needed} |

## Public API

| Type | Description |
|------|-------------|
| `Engine` | Main engine struct |
| `Config` | Configuration struct |
| `Error` | Error type |

## Configuration

```toml
[{module_name}]
enabled = true
timeout_secs = 30
```

## Testing

```bash
cargo test -p voxy-{module_name}
cargo bench -p voxy-{module_name}
```

## Build Order

This module builds in Phase {N} of the VOXY build order.
```

---

## Verification Commands

After creating a new module, run:

```bash
# 1. Check compilation
cargo check -p voxy-{module_name}

# 2. Run tests
cargo test -p voxy-{module_name}

# 3. Check formatting
cargo fmt --check -p voxy-{module_name}

# 4. Run clippy
cargo clippy -p voxy-{module_name} -- -D warnings

# 5. Generate documentation
cargo doc -p voxy-{module_name} --no-deps
```

---

## References

- [Cargo Workspaces](https://doc.rust-lang.org/book/ch14-03-cargo-workspaces.html)
- [Rust API Guidelines](https://rust-lang.github.io/api-guidelines/)

---

## Cross References

- See [01_PROJECT_STRUCTURE.md](01_PROJECT_STRUCTURE.md) for module architecture.
- See [04_CODING_STANDARDS.md](04_CODING_STANDARDS.md) for language conventions.
- See [09_IMPLEMENTATION_CHECKLISTS.md](09_IMPLEMENTATION_CHECKLISTS.md) for per-module requirements.

---

## Best Practices

1. Always start with error types before implementation.
2. Keep public API surface minimal — expose only what is needed.
3. Document every public item with rustdoc comments.
4. Add tests for every public function.
5. Use workspace dependencies for shared crates.

---

## Common Mistakes

| Mistake | Consequence | Prevention |
|---------|-------------|------------|
| Missing error types | Poor error propagation | Implement error.rs first |
| Exposing internals | Tight coupling | Use pub(crate) for internal items |
| No module README | Poor discoverability | Include README.md in every module |
| Hardcoded values | Inflexibility | Use Config for all tunables |

---

## Review Checklist

- [ ] Directory structure matches template.
- [ ] Cargo.toml uses workspace dependencies.
- [ ] lib.rs exports public API cleanly.
- [ ] error.rs defines specific error variants.
- [ ] config.rs uses serde for serialization.
- [ ] engine.rs implements core logic with tests.
- [ ] README.md documents the module.
- [ ] Verification commands all pass.

---

*End of 08_MODULE_TEMPLATE.md*
