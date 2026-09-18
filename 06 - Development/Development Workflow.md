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
The reference environment and revision pins used during dependency normalization and audit testing (all 11 planned verification steps were completed and are presented below in reordered validation categories for readability):

- **Application Repository**:
  - Repo: `DrinkBoooz/cvsu-generators`
  - Branch: `dev`
  - Commit: `bc0ffad1c0199a13a94de9fa328e22b521a4fc9f`
  - Commit Message: `146: strengthen manifest tests with PEP 503 canonicalization and align system prerequisite documentation`
- **Documentation Repository**:
  - Repo: `DrinkBoooz/cvsu-generator_documentation`
  - Branch: `dev`
  - Commit: `4b5dd3d5004d1a0f577cdb6442e0796154b4f02a`
  - Commit Message: `27: synchronize documentation with precise webview2 guidance and environment evidence`
- **Python Runtime**: `Python 3.14.7` (64-bit AMD64)
- **Package Manager**: `pip 26.2.1`
- **Installed-Environment Dependency Health**: `python -m pip check` &rarr; `No broken requirements found.`
- **Forwarding Manifest Validation**: `pip install --dry-run --no-deps -r executable_test/requirements-build.txt` &rarr; pip dry-run direct-requirement / forwarding-manifest validation
- **Playwright Engine**: `playwright==1.63.0`
- **Browser Binary**: Chromium 153.0.8010.12 (`chromium-1243` / `chromium_headless_shell-1243`) at `%LOCALAPPDATA%\ms-playwright\chromium-1243\chrome-win64\chrome.exe`
- **Core Test Telemetry & Categorization**:
  - **Installed Dependency Health (Step 1)**: `python -m pip check` &rarr; `No broken requirements found.`
  - **Selected Unit / Parser / Generator Suite (Step 2)**: 369 selected unit/integration/parser/generator/template/API tests passed, including the 2 newly added dependency-manifest tests (with 2 skipped, 42 deselected). Note: native lifecycle tests, packaged executable tests, desktop integration tests, and UI/Playwright tests are run separately.
  - **Complete Current UI Test Set (Step 3)**: 35 UI/Playwright/resilience tests passed across the complete current UI test set (`pytest tests/test_playwright_e2e.py tests/test_playwright_roster_mapping.py tests/test_playwright_settings_modal.py tests/test_playwright_accessibility.py tests/test_scroll_aware_dock.py tests/test_settings_modal_responsive.py tests/test_theme_transition_perf.py tests/test_ui_asset_resilience.py -q`).
  - **Native Desktop Lifecycle Verification (Step 4)**: Verified via two separate executions:
    - `pytest tests/test_executable_lifecycle.py -v` (2 passed in 0.76s: window closed prevents JS callbacks, lifecycle concurrency race immunity).
    - `pytest tests/test_ui_asset_resilience.py -k "test_desktop_container_live_launch" -v` (1 passed, 7 deselected in 2.64s: live PyWebView container launch and clean exit).
  - **Packaged Executable Smoke Tests (Step 6)**: Filtered run on metadata and process launch (`pytest tests/test_packaged_executable_smoke.py -k "test_packaged_executable_binary_and_pe_metadata or test_packaged_executable_launch_and_cleanup" -v` &rarr; 2 passed, 1 deselected); Authenticode signature status check (`test_packaged_executable_authenticode_signature`) was excluded from this run.
  - **Obsidian Vault Integrity (Step 8)**: 4 passed in 0.07s (`pytest tests/test_obsidian_vault_integrity.py -q`).
  - **Dependency Manifests (Step 7, 9, 10, 11)**: 9 passed in 0.98s (`pytest tests/test_dependency_manifests.py -q`), validating exact direct pins, canonicalized 12-package exclusion (with `pywin32-ctypes` identity preservation), and pip dry-run direct-requirement / forwarding-manifest validation.

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

