---
title: "Agent System Architecture"
tags:
  - cvsu-generator
  - architecture
  - agents
  - governance
status: active
last_modified: 2026-09-13
source_of_truth:
  - AGENTS.md
---

# Agent System Architecture

This document describes how autonomous AI agents and development systems are architected to interact with the CvSU Document Generator project workspace.

Related notes:
- [[CvSU Document Generator MOC]]
- [[Development Workflow]]

---

## 🤖 Governance as Code (`AGENTS.md`)

The primary mechanism for controlling AI agent behavior in this workspace is the `AGENTS.md` file located at the root of the application repository. This file serves as the definitive ruleset for any automated IDE tooling or agentic workflows interacting with the codebase.

### Strict Boundaries & Source of Truth

The agent architecture enforces a strict separation of concerns across two sibling repositories:
1. **Implementation (`CVSU GENERATORS`)**: Code, tests, `.docx`/`.xlsx` templates, and build configuration. This is the executable source of truth.
2. **Knowledge (`cvsu-generator_documentation`)**: Markdown-based architectural notes and user guides. This is the human-readable source of truth.

Agents are strictly instructed **never** to fabricate documentation. Documentation must strictly follow the verified behavior of the actual implementation, tests, and templates.

### Automated Testing Constraints

Agentic changes are constrained by the project's automated test suites:
- **Vault Integrity (`test_obsidian_vault_integrity.py`)**: Agents must successfully update the Obsidian documentation vault in tandem with application changes, or this test will intentionally fail.
- **UI Consistency (`test_ui_consistency.py`)**: Agents must synchronize version numbers across the executable PE metadata (`file_version_info.txt`), the UI badge (`ui.html`), and release notes.

### Predictable Version Control

Agents are bound to strict Git protocols:
- **Lineage Preservation**: Agents are forbidden from automatically merging to `main` and must use explicit `git merge --no-ff dev` when instructed.
- **Commit Formatting**: Agents must adhere to the `<commit_number>: <description based on chat changes>` format, generating sequential commit numbers to ensure an orderly project history.
