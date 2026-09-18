---
title: "Development Workflow"
tags:
  - cvsu-generator
  - development
  - git
  - workflow
status: active
last_modified: 2026-09-18
source_of_truth:
  - AGENTS.md
  - requirements-runtime.txt
  - requirements-test.txt
  - requirements-build.txt
  - requirements.txt
---

# Development Workflow

Guidelines and protocols for developing, committing, branching, and synchronizing versions in the **CvSU Document Generator** repository (`DrinkBoooz/cvsu-generators`).

Related notes:
- [[CvSU Document Generator MOC]]
- [[Testing Strategy]]
- [[Build & Packaging]]
- [[v1.0.1]]

---

## 📦 Dependency Manifest Architecture & Environment Setup

The repository defines an exact direct-dependency specification across four distinct manifests:

| Manifest | Purpose | Direct Packages Included |
| :--- | :--- | :--- |
| **`requirements-runtime.txt`** | Production runtime only | `pywebview==6.2.1`, `pythonnet==3.1.0`, `lxml==6.1.1`, `openpyxl==3.1.5`, `xlrd==2.0.2` |
| **`requirements-test.txt`** | Testing & test development | `-r requirements-runtime.txt`, `pytest==9.1.1`, `python-docx==1.2.0`, `playwright==1.63.0` |
| **`requirements-build.txt`** | PyInstaller standalone packaging | `-r requirements-runtime.txt`, `pyinstaller==6.22.2` |
| **`requirements.txt`** | Aggregate developer setup | `-r requirements-test.txt`, `-r requirements-build.txt` |
| **`executable_test/requirements-build.txt`** | Forwarding manifest for `build.bat` | `-r ../requirements-build.txt` |

### Setting Up a Development Virtual Environment:
```powershell
# 1. Create and activate virtual environment (Python 3.10 - 3.14 x64)
python -m venv .venv
.\.venv\Scripts\activate

# 2. Install aggregate developer dependencies
python -m pip install -r requirements.txt

# 3. Install Playwright Chromium browser binary
python -m playwright install chromium

# 4. Verify environment has no package dependency conflicts
python -m pip check
```

### Verified Development Baseline & Environment Evidence:
The reference environment used during dependency normalization and audit testing:
- **Python Runtime**: `Python 3.14.7` (64-bit AMD64)
- **Package Manager**: `pip 26.2.1`
- **Dependency Health**: `python -m pip check` &rarr; `No broken requirements found.`
- **Playwright Engine**: `playwright==1.63.0`
- **Browser Binary**: Chromium 153.0.8010.12 (`chromium-1243` / `chromium_headless_shell-1243`) at `%LOCALAPPDATA%\ms-playwright\chromium-1243\chrome-win64\chrome.exe`
- **Core Test Commands**:
  - `pytest tests/test_dependency_manifests.py -q`
  - `pytest tests/ -k "not test_playwright" -q`
  - `pytest tests/test_playwright_e2e.py tests/test_playwright_roster_mapping.py tests/test_playwright_settings_modal.py tests/test_playwright_accessibility.py tests/test_scroll_aware_dock.py tests/test_settings_modal_responsive.py tests/test_theme_transition_perf.py tests/test_ui_asset_resilience.py -q`
  - `pytest tests/test_obsidian_vault_integrity.py -q`

---

## 🌿 Git Branching Strategy

- **Development Branch (`dev`)**: All active development, feature implementations, tests, refactors, and chat commits belong strictly on `dev`.
- **Production Branch (`main`)**: The default repository branch.
  - **NEVER Auto-Merge to `main`**: Merging into `main` requires an explicit user command.
  - **Strict No Fast-Forward (`--no-ff`)**: Always use `git merge --no-ff dev` to preserve visual merge nodes and commit lineage in Git graphs.

### Standard Release Merge Sequence:
```bash
git checkout main
git merge --no-ff dev -m "<commit_number>: merge dev into main - <summary of changes>"
git push origin main
git checkout dev
```

---

## 📝 Commit Formatting Rules

Commit messages must strictly follow the sequential numbering scheme:
```text
<commit_number>: <description based on chat changes>
```
- `<commit_number>`: The repository's next sequential commit count (e.g. `git rev-list --count HEAD + 1`).
- Example: `43: dynamically calculate progress bar total steps and update completion telemetry`

---

## 🔄 Version Synchronization Checklist

When releasing a new version:
1. `executable_test/ui.html`: Update `<span class="badge-version">Release vX.Y.Z</span>`.
2. `tests/test_ui_consistency.py`: Update assertion verifying the version badge.
3. `executable_test/README.md`: Update version text and feature lists.
4. `executable_test/file_version_info.txt`: Synchronize `filevers`, `prodvers`, `FileVersion`, `ProductVersion`.
5. `cvsu-generator_documentation/05 - Releases & Changelog/`: Create `vX.Y.Z.md` with release notes.

---

## 🗂️ Coordinated Workspace Architecture & Companion Repositories

The project environment is organized as a **coordinated project workspace** composed of two distinct sibling Git repositories:
- **Application Repository (`CVSU GENERATORS`)**: `C:\Users\danjo\OneDrive\CVSU GENERATORS` (`DrinkBoooz/cvsu-generators`)
- **Documentation Repository (`cvsu-generator_documentation`)**: `C:\Users\danjo\OneDrive\cvsu-generator_documentation` (`DrinkBoooz/cvsu-generator_documentation`)

### Dual-Repository Commit Workflow

Because each component has its own independent Git repository, changes cannot be committed atomically in a single Git commit. Instead, features or refactors spanning both application code and documentation must be committed in their respective repositories with coordinated commit descriptions:

```text
Application Implementation & Tests
    ↓
CVSU GENERATORS repository (branch: dev)

Human-Readable Guides & Vault Notes
    ↓
cvsu-generator_documentation repository (branch: dev)
```

All changes that alter architecture, generators, templates, user workflows, or packaging require updating the corresponding notes in `cvsu-generator_documentation` and validating against `tests/test_obsidian_vault_integrity.py` before finalizing commits.

### Evolution of Governance as Code
The dual-repository protocol was strictly enforced via test automation during the application's development cycle:
- **Formalized Protocol (Commit `b1ad7ff`)**: Introduced the dual-repository documentation governance protocol into `AGENTS.md` and added the vault integrity test.
- **Dynamic Vault Resolution (Commit `7d6ddc2`)**: Replaced hardcoded documentation paths with dynamic sibling-path resolution and `CVSU_VAULT_DIR` environmental overrides, allowing `test_obsidian_vault_integrity.py` to dynamically locate the documentation vault.
- **Strict Enforcement (Commit `88b3839`)**: Removed the permissive `pytest.skipif` logic that silently skipped documentation tests if the vault was absent. The test suite now strictly asserts the existence of the documentation vault and fails if the vault is missing.

