# VOXY — Folder Conventions

| Field | Value |
|-------|-------|
| **Version** | 1.0.0 |
| **Status** | Production-Ready |

---

## Top-Level Structure

```
voxy/
├── .github/          # GitHub configuration
├── docs/             # Documentation (not build-docs)
├── src/              # Source code (20 modules)
├── models/           # Pre-trained model artifacts
├── assets/           # Static resources
├── scripts/          # Build/deploy/test scripts
├── tests/            # Integration and E2E tests
├── tools/            # Development utilities
├── build-docs/       # This documentation repository
├── Cargo.toml        # Workspace root
└── README.md
```

## Module Structure

Every module in src/ follows [08_MODULE_TEMPLATE.md](08_MODULE_TEMPLATE.md).

---

*End of 35_FOLDER_CONVENTIONS.md*