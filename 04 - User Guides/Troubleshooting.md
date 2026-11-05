---
title: "Troubleshooting & Support"
tags:
  - cvsu-generator
  - guide
  - troubleshooting
status: active
last_modified: 2026-11-05
source_of_truth:
  - executable_test/README.md
  - executable_test/sign_exe.ps1
  - executable_test/file_version_info.txt
---

# Troubleshooting & Support

Solutions for common issues, Windows SmartScreen prompts, and file pairing diagnostics.

Related notes:
- [[CvSU Document Generator MOC]]
- [[User Manual]]
- [[Build & Packaging]]
- [[UI Architecture]]

---

## 🛡️ Windows SmartScreen / "Windows protected your PC"

Because this is a specialized institutional application for Cavite State University, Windows SmartScreen may display a warning on initial launch:

### Standard Launch:
1. Click **"More info"** on the blue SmartScreen prompt.
2. Click **"Run anyway"**.

### Verifying Publisher & Authenticode Certificate:
1. Right-click **`CvSU Gen.exe`** &rarr; select **Properties**.
2. Open the **Digital Signatures** tab.
3. Select the signature by **Dan Joseph Ortega** and click **Details**.
4. Open the **Details** tab to view copyright:
   `Copyright © 2026 Dan Joseph Ortega. All rights reserved.`

### Permanent Trust Installation (Optional):
Run **`install_trusted_publisher.bat`** as Administrator once on your computer. This imports the institutional signing certificate into the Windows Trusted Publishers store, permanently eliminating SmartScreen warnings for all future releases.

---

## 🔍 Common Issues & Fixes

### 1. "Missing Student Rosters" or "0 classes matched"
- **Cause**: Roster files were renamed, missing schedule codes, or use custom headers.
- **Fix**: In **Stage 2 (Rosters)**, use the manual dropdown to link the roster to the corresponding schedule code, or click **"Map Columns"** if the file has non-standard header columns.

### 2. "Document generation cancelled"
- **Cause**: The user pressed the "Cancel Generation" button in **Stage 6 (Generate)**.
- **Fix**: Return to the **Generate** stage and click the process button labeled **"Initialize Workflow"** once prerequisites are satisfied. Files already generated remain safely on your disk without corruption or automatic deletion.

### 3. File in use or locked
- **Cause**: An output Word document or Excel sheet is currently open in Microsoft Office.
- **Fix**: Close Microsoft Word or Excel before re-running generation.
- **Under the Hood**: The application engine incorporates a 5-attempt exponential backoff retry policy for reading `.docx` templates (introduced in Commit `9c98065`). This allows the generator to gracefully recover from transient file locks caused by OS indexing or antivirus scans, but a persistent lock by an open Microsoft Word window will still require user intervention.

### 4. "No Classes Selected" Warning during Generation Preflight
- **Cause**: All sections were unchecked in **Stage 3 (Packages)**. While moving between stages via "Continue to Dates" (`activateStage(4)`) remains unblocked, **generation preflight requires at least one selected section**.
- **Fix**: When initializing the workflow without selections (`selectedClasses.length === 0`), generation is blocked with a "No Classes Selected" warning toast, and the application returns you to Stage 3 (`cardStep3`). Check at least one section box (`col-select` using row selector `.item-class-check`) in the class matrix before returning to Stage 6.
