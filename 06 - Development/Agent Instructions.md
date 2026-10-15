---
title: "Agent Instructions"
tags:
  - cvsu-generator
  - development
  - agents
  - rules
status: active
last_modified: 2026-10-15
source_of_truth:
  - AGENTS.md
  - tests/test_golden_parity.py
  - tests/test_deterministic_template_analyzer.py
  - tests/test_dynamic_renderer_resilience.py
---

# Agent Instructions

This note documents the protocols and rules provided to autonomous development agents working on the **CvSU Document Generator** repository.

Related notes:
- [[CvSU Document Generator MOC]]
- [[Development Workflow]]
- [[Agent System Architecture]]
- [[Testing Strategy]]

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

### 7. Modernized Dynamic Template Protocol
With the completion of the **Self-Analyzing Dynamic Template Engine**, agents should no longer author manual Python subclasses with hardcoded table indices (e.g. `tables[1]`) or fixed coordinate tuples:
- **Automatic Structural Introspection**: Ingesting new Word (`.docx`) or Excel (`.xlsx`) templates is handled via `DeterministicTemplateAnalyzer` (`modules/parsers/template_analyzer.py`) and `DocxGridTable` (`modules/parsers/docx_grid_table.py`), which detect semantic slots (`cell_right`, `cell_below`, `colon_value`, `placeholder`) and repeating table topologies dynamically.
- **Dynamic Style-Preserving Rendering**: Generation executes via `DynamicDocumentRenderer` (`modules/generators/dynamic_renderer.py`) or `ConfigurableDocumentGenerator`, preserving original OpenXML formatting, native `<w:numPr>` list numbering, and applying the unified student name font scaling ladder.
- **Testing Verification**: Any new or updated template must pass:
  - `tests/test_golden_parity.py` (proves bit-for-bit parity against reference baselines).
  - `tests/test_deterministic_template_analyzer.py` (validates offline slot introspection in <20ms).
  - `tests/test_dynamic_renderer_resilience.py` (proves immunity to structural decoy tables and layout shifts).

### 8. Verification Loop & Self-Evaluation Gate
Before completing any task, agents must execute the mandatory verification loop:
- Run `pytest tests/test_obsidian_vault_integrity.py tests/test_manifest_integrity.py tests/test_ast_rules.py tests/test_dependency_manifests.py`.
- Run `python scripts/test_orchestrator.py --fast` (Python 3.14 baseline) verifying 100% pass rate across all layers.
- Apply the structured 5-axis self-evaluation scorecard (Accuracy, Completeness, Clarity, Actionability, Conciseness) per `.agents/skills/agent-self-evaluation/SKILL.md`.

