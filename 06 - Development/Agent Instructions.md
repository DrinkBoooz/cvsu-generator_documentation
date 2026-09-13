---
title: "Agent Instructions"
tags:
  - cvsu-generator
  - development
  - agents
  - rules
status: active
last_modified: 2026-09-13
source_of_truth:
  - AGENTS.md
---

# Agent Instructions

This note documents the protocols and rules provided to autonomous development agents working on the **CvSU Document Generator** repository.

Related notes:
- [[CvSU Document Generator MOC]]
- [[Development Workflow]]

---

## 🤖 Agent Execution Rules

Agents working on the `CVSU GENERATORS` project must adhere to a strict set of global rules enforced via the workspace `AGENTS.md` rule file.

### 1. Tests Location
All automated tests, scripts, and fixtures must remain in `tests/`. No test scripts are permitted in the project root.

### 2. Git Commit Formatting
Agents must strictly use `<commit_number>: <description>` for every commit, ensuring an unbroken sequential history for the project.

### 3. Git Branching & Merging
All work occurs on `dev`. Merging to `main` must never be automated; it requires explicit user instruction and must use `--no-ff`.

### 4. Version Numbering & Synchronization
When a version bump occurs, agents must synchronize the version number across the UI (`ui.html`), PE build metadata (`file_version_info.txt`), automated tests (`test_ui_consistency.py`), the README, and release notes in the documentation vault.

### 5. Obsidian Documentation Protocol
The Obsidian vault (`cvsu-generator_documentation`) is the primary human-readable knowledge base. Agents must update it whenever architectural, generator, template, or workflow changes are implemented. The vault must reflect verified code behavior, not fabricated logic.

### 6. Executable Packaging & Code Signing
Agents must ensure the Windows PE build correctly incorporates Authenticode code signing using `sign_exe.ps1` and embeds `file_version_info.txt`.

### 7. Adding New Templates Protocol
When adding new `.docx` or `.xlsx` templates, agents must place them in the correct folder, inspect template structure, write a subclass implementation, register it in the `GeneratorFactory`, update tests, and document the behavior in the vault.
