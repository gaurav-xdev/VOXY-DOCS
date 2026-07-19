# VOXY — Engineering Rules

| Field | Value |
|-------|-------|
| **Version** | 1.0.0 |
| **Status** | Production-Ready |
| **Last Updated** | 2026-07-17 |
| **Author** | VOXY Engineering Team |
| **Classification** | Internal — Governance Foundation |

---

## Purpose

This document defines the non-negotiable engineering rules, constraints, and policies that govern all development in the VOXY project. These rules are enforced by CI gates, code review, and architectural review. No exception is permitted without an approved Architecture Decision Record (ADR).

---

## Scope

Covers:
- Security rules
- Performance rules
- Reliability rules
- Privacy rules
- Accessibility rules
- Compatibility rules
- Licensing rules

Does not cover:
- Coding style (see [04_CODING_STANDARDS.md](04_CODING_STANDARDS.md))
- Workflow procedures (see [06_DEVELOPER_WORKFLOW.md](06_DEVELOPER_WORKFLOW.md))
- Technology selections (see [03_TECH_STACK.md](03_TECH_STACK.md))

---

## Audience

- All engineers (rules are mandatory)
- Security team
- Compliance officers
- Architecture Review Board
- AI Coding Agents (must follow as constraints)

---

## Rule Categories

### Security Rules (SEC-*) — CRITICAL

| ID | Rule | Enforcement | Exception Process |
|----|------|-------------|-------------------|
| **SEC-001** | All network communication must use TLS 1.3 or higher. | CI static analysis | ADR + Security review |
| **SEC-002** | No secrets (keys, tokens, passwords) in source code. | git-secrets + CI scan | None — use secret management |
| **SEC-003** | All user data must be encrypted at rest using AES-256-GCM. | Code review + audit | ADR + Crypto review |
| **SEC-004** | Credential storage must use Windows Credential Manager or DPAPI. | CI lint + runtime check | ADR |
| **SEC-005** | All file I/O outside the app directory requires explicit user consent. | Runtime permission check | ADR + Legal review |
| **SEC-006** | No execution of user-provided code without sandboxing. | Code review + static analysis | ADR + Security review |
| **SEC-007** | All dependencies must pass `cargo audit` with zero critical/high vulnerabilities. | CI gate | ADR + Security review |
| **SEC-008** | Input from audio transcription must be sanitized before use in file paths or commands. | Code review + fuzzing | ADR |
| **SEC-009** | Plugin system must run in a separate process with restricted capabilities. | Architecture review | None |
| **SEC-010** | Update packages must be signed and signature verified before installation. | CI + runtime verification | None |

### Performance Rules (PERF-*) — HIGH

| ID | Rule | Enforcement | Exception Process |
|----|------|-------------|-------------------|
| **PERF-001** | Voice command E2E latency must not exceed 500ms for simple commands. | CI benchmark gate | ADR + Performance review |
| **PERF-002** | Wake word detection must complete within 100ms of audio arrival. | CI benchmark gate | ADR |
| **PERF-003** | UI thread must not block for more than 16ms (60fps). | Profiling + CI | ADR |
| **PERF-004** | Memory usage for AI models must not exceed 8GB on target hardware. | CI benchmark gate | ADR |
| **PERF-005** | Audio capture latency must not exceed 10ms (WASAPI event-driven). | CI benchmark gate | ADR |
| **PERF-006** | Background tasks must yield control every 50ms to prevent starvation. | Code review | ADR |
| **PERF-007** | Model loading must happen off the main thread. | CI lint + code review | None |
| **PERF-008** | Unused model weights must be unloaded after 5 minutes of inactivity. | Runtime monitoring | ADR |

### Reliability Rules (REL-*) — HIGH

| ID | Rule | Enforcement | Exception Process |
|----|------|-------------|-------------------|
| **REL-001** | All panics must be converted to structured errors before reaching the user. | CI lint (`forbid(panic)`) | None |
| **REL-002** | Every async operation must have a timeout. | Code review + runtime check | ADR |
| **REL-003** | All external service calls must implement circuit breaker pattern. | Code review | ADR |
| **REL-004** | State changes must be logged with before/after snapshots for audit. | Code review | ADR + Legal review |
| **REL-005** | Graceful degradation: if AI model fails, fall back to rule-based processing. | Integration tests | ADR |
| **REL-006** | Application must recover from crash within 5 seconds on restart. | E2E tests | ADR |
| **REL-007** | No data loss on unexpected termination: all state changes are atomic. | Code review + tests | None |
| **REL-008** | Update rollback must restore previous version within 30 seconds. | E2E tests | None |

### Privacy Rules (PRIV-*) — CRITICAL

| ID | Rule | Enforcement | Exception Process |
|----|------|-------------|-------------------|
| **PRIV-001** | Voice recordings are processed locally; no audio leaves the device. | Architecture review + network monitoring | None |
| **PRIV-002** | Transcripts are not persisted unless explicitly enabled by user. | Runtime check + code review | ADR + Legal review |
| **PRIV-003** | Telemetry contains no personally identifiable information (PII). | Data classification review | ADR + Legal review |
| **PRIV-004** | User must be notified before any data leaves the device (cloud fallback). | UI review + runtime check | ADR + Legal review |
| **PRIV-005** | All data retention policies must be configurable by the user. | Feature review | ADR |
| **PRIV-006** | GDPR/CCPA compliance: data export and deletion must be supported. | Legal review + tests | None |

### Accessibility Rules (A11Y-*) — HIGH

| ID | Rule | Enforcement | Exception Process |
|----|------|-------------|-------------------|
| **A11Y-001** | All UI elements must have accessible names and descriptions. | Accessibility audit | ADR |
| **A11Y-002** | Keyboard navigation must be possible for all features. | Manual testing | ADR |
| **A11Y-003** | Color contrast must meet WCAG 2.1 AA standards (4.5:1). | Automated testing | ADR |
| **A11Y-004** | Screen reader compatibility for all status and feedback messages. | Manual testing + NVDA | ADR |
| **A11Y-005** | Voice commands must have text equivalents for non-voice users. | Feature review | None |

### Compatibility Rules (COMP-*) — HIGH

| ID | Rule | Enforcement | Exception Process |
|----|------|-------------|-------------------|
| **COMP-001** | Minimum supported OS: Windows 10 22H2 (build 19045). | CI testing on Windows 10 | ADR |
| **COMP-002** | Application must function without internet connectivity. | Network-disabled tests | None |
| **COMP-003** | All Win32 API calls must handle API absence on older Windows versions. | Runtime checks | ADR |
| **COMP-004** | MSIX package must install on both Windows 10 and Windows 11. | CI packaging tests | None |
| **COMP-005** | Application must degrade gracefully on systems without GPU acceleration. | CI tests on CPU-only VMs | ADR |

### Licensing Rules (LIC-*) — CRITICAL

| ID | Rule | Enforcement | Exception Process |
|----|------|-------------|-------------------|
| **LIC-001** | All dependencies must be MIT, Apache-2.0, BSD-3-Clause, or equivalent. | `cargo deny` + legal review | ADR + Legal review |
| **LIC-002** | No GPL, AGPL, or copyleft dependencies in any module. | `cargo deny` + legal review | None |
| **LIC-003** | All third-party code must have attribution in NOTICE file. | CI check | None |
| **LIC-004** | Model weights must have compatible licenses (CC-BY, Apache-2.0, or proprietary with rights). | Legal review | ADR + Legal review |

---

## Rule Violation Severity Matrix

| Severity | Action | Timeline |
|----------|--------|----------|
| **CRITICAL** | Block merge, escalate to Security/Architecture Review Board | Immediate |
| **HIGH** | Block merge, require ADR for exception | Before release |
| **MEDIUM** | Require remediation in next sprint | Within 2 weeks |
| **LOW** | Track in backlog, address in next quarter | Within 3 months |

---

## Enforcement Mechanisms

### CI Gates

```yaml
# .github/workflows/ci.yml (excerpt)
jobs:
  security-scan:
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v4
      - name: Cargo Audit
        run: cargo audit --deny warnings
      - name: Cargo Deny
        run: cargo deny check
      - name: Secret Scan
        uses: trufflesecurity/trufflehog@main
        with:
          path: ./
          base: main
      - name: Clippy Security Lints
        run: cargo clippy -- -D warnings -W clippy::unwrap_used -W clippy::expect_used

  performance-gate:
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build Release
        run: cargo build --release
      - name: Run Benchmarks
        run: cargo bench --workspace
      - name: Check Latency Budgets
        run: python scripts/ci/check_latency_budgets.py

  compliance-check:
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v4
      - name: License Check
        run: cargo deny check licenses
      - name: Accessibility Audit
        run: scripts/ci/accessibility-audit.ps1
```

### Code Review Requirements

Every pull request must have:
- [ ] Security review for SEC-* rule compliance
- [ ] Performance review for PERF-* rule compliance
- [ ] Reliability review for REL-* rule compliance
- [ ] Privacy review for PRIV-* rule compliance (if user-facing)
- [ ] Accessibility review for A11Y-* rule compliance (if UI changes)

### Runtime Enforcement

```rust
// Example: Runtime privacy enforcement
pub async fn process_voice_command(audio: AudioBuffer) -> Result<CommandResult> {
    // PRIV-001: Process locally
    let transcript = local_asr_engine.transcribe(&audio).await?;

    // PRIV-002: Do not persist unless configured
    if config.persist_transcripts {
        secure_store.append_transcript(&transcript).await?;
    }

    // PRIV-004: Notify before cloud fallback
    if !local_nlu.can_handle(&transcript) && config.allow_cloud_fallback {
        ui.show_cloud_fallback_notification().await?;
        if !ui.user_confirmed().await? {
            return Err(Error::UserDeclinedCloud);
        }
    }

    // ...
}
```

---

## Exception Process

1. **Identify the rule** being violated and document the business/technical justification.
2. **Draft an ADR** using the template in [29_ADR_TEMPLATE.md](29_ADR_TEMPLATE.md).
3. **Submit for review** to the Architecture Review Board.
4. **Security/Compliance review** for SEC, PRIV, LIC rules.
5. **Board vote:** Majority approval required for CRITICAL rules; simple majority for HIGH.
6. **Document the exception** in the decision log ([28_DECISION_LOG_TEMPLATE.md](28_DECISION_LOG_TEMPLATE.md)).
7. **Set expiration date** for the exception (default: 6 months).
8. **Implement compensating controls** if the exception increases risk.

---

## Engineering Notes

### Rule Origin

These rules are derived from:
- Microsoft SDL (Security Development Lifecycle)
- OWASP Top 10
- GDPR Article 25 (Data Protection by Design)
- WCAG 2.1 Guidelines
- Windows App Certification Requirements
- Enterprise customer security questionnaires

### Rule Evolution

Rules are reviewed quarterly by the Architecture Review Board. Proposals for new rules or modifications follow the RFC process ([30_RFC_TEMPLATE.md](30_RFC_TEMPLATE.md)).

---

## References

- [Microsoft SDL](https://www.microsoft.com/en-us/securityengineering/sdl)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [GDPR Article 25](https://gdpr-info.eu/art-25-gdpr/)
- [WCAG 2.1](https://www.w3.org/WAI/WCAG21/quickref/)
- [Windows App Certification Kit](https://learn.microsoft.com/en-us/windows/uwp/debug-test-perf/windows-app-certification-kit)

---

## Cross References

- See [04_CODING_STANDARDS.md](04_CODING_STANDARDS.md) for code-level conventions.
- See [06_DEVELOPER_WORKFLOW.md](06_DEVELOPER_WORKFLOW.md) for review workflow.
- See [28_DECISION_LOG_TEMPLATE.md](28_DECISION_LOG_TEMPLATE.md) for exception tracking.
- See [29_ADR_TEMPLATE.md](29_ADR_TEMPLATE.md) for exception documentation.
- See [30_RFC_TEMPLATE.md](30_RFC_TEMPLATE.md) for rule change proposals.

---

## Best Practices

1. **Assume rules apply** unless explicitly exempted.
2. **Question everything** that seems to violate a rule — better to ask than apologize.
3. **Document compensating controls** when rules create friction.
4. **Review rules quarterly** with the Architecture Review Board.
5. **Train new team members** on rules during onboarding.

---

## Common Mistakes

| Mistake | Consequence | Prevention |
|---------|-------------|------------|
| Assuming a rule doesn't apply | Security vulnerability, compliance failure | Check rule scope carefully |
| Bypassing CI gates locally | Undetected violations | Pre-commit hooks enforce gates |
| Not documenting exceptions | Audit failure, knowledge loss | Mandatory ADR for all exceptions |
| Ignoring MEDIUM/LOW rules | Technical debt accumulation | Sprint planning includes rule remediation |

---

## Review Checklist

- [ ] All rules have unique IDs and clear descriptions.
- [ ] Enforcement mechanisms are implemented in CI.
- [ ] Exception process is documented and followed.
- [ ] Rule severity matrix is understood by all engineers.
- [ ] Quarterly review is scheduled.
- [ ] New team members are trained on rules within first week.

---

*End of 05_ENGINEERING_RULES.md*
