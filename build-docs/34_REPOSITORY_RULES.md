# VOXY — Repository Rules

| Field | Value |
|-------|-------|
| **Version** | 1.0.0 |
| **Status** | Production-Ready |

---

## Branch Protection

| Branch | Required Reviews | Required Checks | Force Push |
|--------|-----------------|-----------------|------------|
| main | 2 | All CI gates | Forbidden |
| develop | 1 | Lint + Build + Test | Forbidden |
| release/* | 2 | All gates + Security | Forbidden |
| hotfix/* | 1 (expedited) | All gates | Forbidden |

## File Patterns

| Pattern | Reviewers Required |
|---------|-------------------|
| src/security/** | Security team + 1 |
| src/ui/** | UI team + 1 |
| Cargo.toml | Tech Lead |
| .github/workflows/** | DevOps + 1 |

## Commit Signing

All commits must be GPG-signed. Unsigned commits are rejected.

---

*End of 34_REPOSITORY_RULES.md*