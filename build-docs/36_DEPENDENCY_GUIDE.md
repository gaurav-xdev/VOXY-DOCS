# VOXY — Dependency Guide

| Field | Value |
|-------|-------|
| **Version** | 1.0.0 |
| **Status** | Production-Ready |

---

## Adding Dependencies

1. Check if already in workspace Cargo.toml
2. Evaluate against [03_TECH_STACK.md](03_TECH_STACK.md) criteria
3. Run cargo-deny license check
4. Add to workspace [workspace.dependencies]
5. Reference in module Cargo.toml with { workspace = true }
6. Document in module README

## Prohibited Dependencies

| Category | Reason |
|----------|--------|
| GPL/AGPL | License incompatibility |
| Unmaintained (>2 years) | Security risk |
| No Windows support | Platform requirement |
| Heavy runtime dependencies | Offline requirement |

## Vulnerability Response

| Severity | Response Time | Action |
|----------|--------------|--------|
| Critical | 24 hours | Emergency patch release |
| High | 72 hours | Scheduled patch |
| Medium | Next sprint | Planned update |
| Low | Next quarter | Backlog |

---

*End of 36_DEPENDENCY_GUIDE.md*