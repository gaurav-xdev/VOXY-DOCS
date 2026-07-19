# VOXY — AI Agent Playbook

| Field | Value |
|-------|-------|
| **Version** | 1.0.0 |
| **Status** | Production-Ready |
| **Last Updated** | 2026-07-17 |
| **Author** | VOXY Engineering Team |
| **Classification** | Internal — AI Agent Operations |

---

## Purpose

This document is the single source of truth for AI coding agents (Claude Code, Codex, Cursor, Gemini CLI, Copilot, RooCode, Aider, etc.) implementing features in the VOXY codebase. It provides context, constraints, patterns, and verification steps that agents must follow.

---

## Scope

Covers:
- Agent context initialization
- Module implementation patterns
- Code generation constraints
- Testing requirements for agent-generated code
- Verification checklist for agent outputs
- Common agent failure modes and prevention

---

## Audience

- AI Coding Agents (Claude Code, Codex, Cursor, Gemini CLI, Copilot, RooCode, Aider)
- Engineers using AI agents for VOXY development
- Tool authors integrating with VOXY

---

## Agent Context Initialization

Before generating any code, the agent must:

1. **Read 00_READ_FIRST.md** — Understand project context and core principles
2. **Read 01_PROJECT_STRUCTURE.md** — Understand module architecture
3. **Read 03_TECH_STACK.md** — Know available technologies
4. **Read 04_CODING_STANDARDS.md** — Follow language conventions
5. **Read 05_ENGINEERING_RULES.md** — Respect non-negotiable constraints
6. **Read 08_MODULE_TEMPLATE.md** — Use standard module structure
7. **Read 26_NAMING_CONVENTIONS.md** — Apply correct naming

### Context Window Priority

If context is limited, prioritize in this order:
1. The target module's README.md
2. 04_CODING_STANDARDS.md (relevant language section)
3. 05_ENGINEERING_RULES.md (relevant rule category)
4. 01_PROJECT_STRUCTURE.md (module dependencies)
5. 08_MODULE_TEMPLATE.md (scaffolding)

---

## Module Implementation Patterns

### New Module Creation

When creating a new module, follow this exact sequence:

```rust
// 1. Define the module structure per 08_MODULE_TEMPLATE.md
// 2. Implement error types first
// 3. Define public API (traits and structs)
// 4. Implement core logic
// 5. Add unit tests
// 6. Add documentation
// 7. Run verification commands
```

### Modifying Existing Modules

```rust
// 1. Read the module's README.md and existing code
// 2. Identify the specific component to modify
// 3. Check for dependent modules (01_PROJECT_STRUCTURE.md)
// 4. Implement changes following 04_CODING_STANDARDS.md
// 5. Add/update tests
// 6. Run: cargo test, cargo clippy, cargo fmt
// 7. Update CHANGELOG.md if user-facing change
```

---

## Code Generation Constraints

### Mandatory Constraints

| Constraint | Enforcement | Failure Mode |
|------------|-------------|--------------|
| No unwrap() or expect() in production paths | Clippy lint unwrap_used | Panic in production |
| All public items must have doc comments | CI gate | Poor API documentation |
| Error types must implement std::error::Error | Code review | Incompatible error handling |
| Async I/O must use Tokio | Code review | Thread pool exhaustion |
| No unsafe without SAFETY comment + ADR reference | Code review + CI | Undefined behavior |
| All functions must have tests | CI gate | Untested code in production |
| Dependencies must be in workspace Cargo.toml | CI gate | Version conflicts |

### Prohibited Patterns

```rust
// NEVER generate this:
let result = some_operation().unwrap(); // PANIC RISK

// NEVER generate this:
unsafe { *raw_ptr = value; } // NO SAFETY COMMENT

// NEVER generate this:
std::thread::sleep(Duration::from_millis(100)); // BLOCKING IN ASYNC

// NEVER generate this:
let data = reqwest::get(url).await?.text().await?; // NO TIMEOUT

// NEVER generate this:
fn process(data: &str) -> String { // NO ERROR TYPE
    // ...
}
```

### Required Patterns

```rust
// ALWAYS generate this:
let result = some_operation().context("operation failed")?;

// ALWAYS generate this:
/// SAFETY: The pointer is valid because...
unsafe { /* ... */ }

// ALWAYS generate this:
tokio::time::timeout(Duration::from_secs(5), async_operation()).await??

// ALWAYS generate this:
#[derive(thiserror::Error, Debug)]
pub enum ProcessError {
    #[error("invalid input: {0}")]
    InvalidInput(String),
}

pub fn process(data: &str) -> Result<String, ProcessError> {
    // ...
}
```

---

## Testing Requirements for Agent-Generated Code

### Minimum Test Coverage

| Code Type | Minimum Tests Required |
|-----------|----------------------|
| Public function | 1 happy path + 1 error path |
| Async function | 1 success + 1 timeout/cancellation |
| Trait implementation | All trait methods |
| Error type | All variants constructed |
| Unsafe block | Fuzz test or property-based test |

### Test Template

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn function_name_happy_path() {
        // Arrange
        let input = create_test_input();

        // Act
        let result = function_under_test(input);

        // Assert
        assert!(result.is_ok());
        assert_eq!(result.unwrap(), expected_output);
    }

    #[test]
    fn function_name_error_path() {
        // Arrange
        let invalid_input = create_invalid_input();

        // Act
        let result = function_under_test(invalid_input);

        // Assert
        assert!(matches!(result, Err(ExpectedError::Variant)));
    }

    #[tokio::test]
    async fn async_function_timeout() {
        let result = tokio::time::timeout(
            Duration::from_millis(100),
            slow_function()
        ).await;

        assert!(result.is_err()); // Should timeout
    }
}
```

---

## Verification Checklist for Agent Outputs

Before submitting agent-generated code:

- [ ] cargo check passes without errors
- [ ] cargo clippy -- -D warnings passes
- [ ] cargo fmt --check passes
- [ ] cargo test passes (all new tests)
- [ ] cargo doc --no-deps generates without warnings
- [ ] No unwrap() or expect() in production paths
- [ ] All public items have doc comments
- [ ] Error types are specific and implement std::error::Error
- [ ] Async code uses Tokio patterns correctly
- [ ] Unsafe code has SAFETY comments
- [ ] No new dependencies without workspace declaration
- [ ] CHANGELOG.md updated if user-facing

---

## Common Agent Failure Modes and Prevention

| Failure Mode | Cause | Prevention |
|-------------|-------|------------|
| Hallucinated APIs | Agent invents non-existent methods | Always check crate docs; use only documented APIs |
| Missing error handling | Agent optimizes for brevity | Enforce Result return types; never return unwrapped values |
| Blocking in async context | Agent does not understand Tokio | Review all I/O operations; use spawn_blocking for CPU work |
| Incorrect lifetimes | Agent struggles with Rust borrow checker | Use Arc<str> or owned types for cross-thread data |
| Missing tests | Agent forgets test requirements | Use test template; verify coverage before submission |
| Wrong module structure | Agent does not follow template | Reference 08_MODULE_TEMPLATE.md explicitly |
| Version conflicts | Agent uses wrong dependency version | Check workspace Cargo.toml; use workspace dependencies |

---

## Agent-Specific Instructions

### Claude Code
- Use /read to load context documents before coding
- Use /terminal to run verification commands
- Use /edit for precise modifications

### Codex / Cursor
- Include context documents in the chat before requesting code
- Request verification commands after code generation
- Use Composer for multi-file changes

### Gemini CLI
- Use --context flag with relevant document paths
- Request structured output with verification steps

### Aider
- Add context documents to the chat with /add
- Use /test to run verification after changes

---

## References

- [04_CODING_STANDARDS.md](04_CODING_STANDARDS.md)
- [05_ENGINEERING_RULES.md](05_ENGINEERING_RULES.md)
- [08_MODULE_TEMPLATE.md](08_MODULE_TEMPLATE.md)

---

## Cross References

- See [04_CODING_STANDARDS.md](04_CODING_STANDARDS.md) for code conventions.
- See [05_ENGINEERING_RULES.md](05_ENGINEERING_RULES.md) for non-negotiable constraints.
- See [08_MODULE_TEMPLATE.md](08_MODULE_TEMPLATE.md) for module scaffolding.
- See [09_IMPLEMENTATION_CHECKLISTS.md](09_IMPLEMENTATION_CHECKLISTS.md) for per-module requirements.

---

## Best Practices

1. Always load context before generating code.
2. Generate error types before implementation logic.
3. Write tests alongside implementation, not after.
4. Run verification commands after every significant change.
5. When uncertain, ask for clarification rather than guessing.

---

## Common Mistakes

| Mistake | Consequence | Prevention |
|---------|-------------|------------|
| Generating code without context | Incorrect assumptions, wrong patterns | Mandatory context loading |
| Skipping verification | Broken builds, CI failures | Automated verification checklist |
| Ignoring module boundaries | Tight coupling, circular dependencies | Follow 01_PROJECT_STRUCTURE.md |
| Over-engineering | Unnecessary complexity | Start simple, iterate |

---

## Review Checklist

- [ ] Agent playbook is referenced in all AI agent sessions.
- [ ] Context initialization steps are followed.
- [ ] Code generation constraints are enforced.
- [ ] Verification checklist is completed for all outputs.
- [ ] Failure modes are monitored and addressed.

---

*End of 07_AI_AGENT_PLAYBOOK.md*
