---
title: "Troubleshooting & Support"
tags:
  - cvsu-generator
  - guide
  - troubleshooting
status: active
last_modified: 2026-09-13
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
- **Cause**: Roster files were renamed or do not include the schedule code.
- **Fix**: Keep original portal filename: `{Course/Sec} List of Students for {ScheduleCode}-{Subject}.xlsx`.

### 2. "Document generation cancelled"
- **Cause**: The user pressed the "Cancel Generation" button in Step 6.
- **Fix**: Re-click "Initialize Workflow" to start generation again. Files already generated remain safely on disk.

### 3. File in use or locked
- **Cause**: An output Word document or Excel sheet is currently open in Microsoft Office.
- **Fix**: Close Microsoft Word or Excel before re-running generation. 
- **Under the Hood**: The application engine incorporates a 5-attempt exponential backoff retry policy for reading `.docx` templates (introduced in Commit `9c98065`). This allows the generator to gracefully recover from transient file locks caused by OS indexing or antivirus scans, but a persistent lock by an open Microsoft Word window will still require user intervention.
