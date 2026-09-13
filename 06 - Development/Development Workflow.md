---
title: "Development Workflow"
tags:
  - cvsu-generator
  - development
  - git
  - workflow
status: active
last_modified: 2026-09-13
source_of_truth:
  - AGENTS.md
---

# Development Workflow

Guidelines and protocols for developing, committing, branching, and synchronizing versions in the **CvSU Document Generator** repository (`DrinkBoooz/cvsu-generators`).

Related notes:
- [[CvSU Document Generator MOC]]
- [[Testing Strategy]]
- [[Build & Packaging]]
- [[v1.0.1]]

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

