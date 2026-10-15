---
title: "Agent System Architecture"
tags:
  - cvsu-generator
  - architecture
  - agents
  - governance
status: active
last_modified: 2026-10-15
source_of_truth:
  - AGENTS.md
  - tests/test_golden_parity.py
  - tests/test_deterministic_template_analyzer.py
  - tests/test_dynamic_renderer_resilience.py
  - tests/test_obsidian_vault_integrity.py
---

# Agent System Architecture

This document describes how autonomous AI agents and development systems are architected to interact with the CvSU Document Generator project workspace.

Related notes:
- [[CvSU Document Generator MOC]]
- [[Development Workflow]]
- [[Agent Instructions]]
- [[Testing Strategy]]

---

## 🤖 Governance as Code (`AGENTS.md`)

The primary mechanism for controlling AI agent behavior in this workspace is the `AGENTS.md` file located at the root of the application repository. This file serves as the definitive ruleset for any automated IDE tooling or agentic workflows interacting with the codebase.

### Strict Boundaries & Source of Truth

The agent architecture enforces a strict separation of concerns across two sibling repositories:
1. **Implementation (`CVSU GENERATORS`)**: Code, tests, `.docx`/`.xlsx` templates, and build configuration. This is the executable source of truth.
2. **Knowledge (`cvsu-generator_documentation`)**: Markdown-based architectural notes and user guides. This is the human-readable source of truth.

Agents are strictly instructed **never** to fabricate documentation. Documentation must strictly follow the verified behavior of the actual implementation, tests, and templates.

---

## 🎯 Pruned Python/Web-Focused Agent Configuration (`.agents/`)

The repository's `.agents/` configuration underwent a comprehensive governance cleanup:
- **Pruned Unused Language Tooling**: Removed hundreds of superfluous multi-language rules and skills (such as C++, Rust, Go, Ruby, PHP, and Java) that previously polluted agent context windows.
- **Focused Execution Domain**: Retained and optimized only the core skills and workflows relevant to the application's actual tech stack:
  - **Python Core**: Modern typing, AST governance, and `pytest` testing patterns (`python-testing-patterns`).
  - **Frontend / PyWebView**: Vanilla HTML5/CSS3/JS, Apple HIG, and accessibility testing (`modern-web-guidance`, `browser-qa`).
  - **Workflow Governance**: Verification loops (`verification-loop`), self-evaluations (`agent-self-evaluation`), living documentation governance (`living-docs-governance`), and architecture decision records (`architecture-decision-records`).

---

## 🧪 Automated Testing Constraints & Verification Gates

Agentic changes are constrained by the project's automated test suites:
- **Vault Integrity (`test_obsidian_vault_integrity.py`)**: Agents must successfully update the Obsidian documentation vault in tandem with application changes, or this test will intentionally fail.
- **Manifest & AST Governance (`test_manifest_integrity.py`, `test_ast_rules.py`)**: Enforces that all test files are registered in `scripts/test_orchestrator.py` and that no hardcoded cell coordinates or direct recipe constructions violate architectural boundaries.
- **Golden-Master Parity (`test_golden_parity.py`)**: Verifies 100% byte-accurate rendering and OpenXML run parity against known reference outputs across all CEIT forms and attendance sheets.
- **Deterministic Analyzer Introspection (`test_deterministic_template_analyzer.py`)**: Validates that template introspection executes in under 20ms with 100% precision across metadata slots, table topologies, and explicit tags.
- **Perturbation Resilience (`test_dynamic_renderer_resilience.py`)**: Asserts that `DynamicDocumentRenderer` correctly binds and generates documents even when templates suffer adversarial perturbations (decoy tables, warning notes, shifted row orders).
- **UI Consistency (`test_ui_consistency.py`)**: Agents must synchronize version numbers across the executable PE metadata (`file_version_info.txt`), the UI badge (`ui.html`), and release notes.

### Predictable Version Control

Agents are bound to strict Git protocols:
- **Lineage Preservation**: Agents are forbidden from automatically merging to `main` and must use explicit `git merge --no-ff dev` when instructed.
- **Commit Formatting**: Agents must adhere to the `<commit_number>: <description based on chat changes>` format, generating sequential commit numbers to ensure an orderly project history.

