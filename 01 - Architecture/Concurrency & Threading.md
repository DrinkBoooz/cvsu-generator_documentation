---
title: "Concurrency & Threading"
tags:
  - cvsu-generator
  - architecture
  - concurrency
  - threading
  - pdf-pipeline
status: active
last_modified: 2026-10-27
source_of_truth:
  - modules/services/orchestrator.py
  - modules/common/pdf_exporter.py
  - executable_test/native/dnd.py
---

# Concurrency & Threading

This document outlines the threading model, concurrency safety, and cancellation engine of the CvSU Document Generator application.

Related notes:
- [[CvSU Document Generator MOC]]
- [[System Architecture]]
- [[Orchestrator Lifecycle]]
- [[PyWebView Bridge]]

---

## 🧵 Threading Architecture

The application operates in a multi-threaded PyWebView environment, primarily separating the UI event loop from the heavy document processing tasks to prevent UI freezing.

### 1. Main UI Thread
- Runs the Chromium Webview rendering engine.
- Handles UI state updates and animations.

### 2. Background Processing Thread
- Executed via PyWebView's background window execution or explicitly spawned threads (`threading.Thread`).
- Handles I/O bound operations like parsing `.xlsx` rosters and generating `.docx`/`.xlsx` output files.

### 3. Native Windows Threading
- **Native OLE Drag and Drop**: Introduced in Commit `c9e2c00`, the application uses `pythonnet` to implement `IDropTarget` for native Windows file drag-and-drop. This avoids legacy Base64 JavaScript file drops and directly resolves paths at the OS level. Thread safety was overhauled to safely interface `IDropTarget` with the Python backend.

### 4. Pipelined PDF Export Worker Thread (`ConcurrentPdfPipeline`)
- **Producer-Consumer Overlap**: When "Export as PDF" is toggled, `ConcurrentPdfPipeline` initializes a dedicated daemon worker thread (`PdfExportPipelineWorker`).
- **STA COM Isolation**: Microsoft Word automation on Windows requires a Single-Threaded Apartment (STA). By isolating Word COM conversion into a dedicated worker thread with `pythoncom.CoInitialize()`, Python document generation (`python-docx`, `openpyxl`) on the main generation thread runs fully concurrently with active Word COM conversions.
- **Queued Submission**: As each CEIT form or monthly attendance sheet is written to disk, it is immediately enqueued via `pdf_pipeline.submit()`. Word converts queued files in the background while subsequent sections and classes are being processed.

---

## 📑 Localized PDF Merging & Bundling Mechanics

Once all document generation completes, `orchestrator.py` drains and finishes `ConcurrentPdfPipeline` and executes high-speed vector-preserving PDF merges using `pypdf.PdfWriter`:

1. **Chronological Attendance Bundles**:
   - Monthly attendance PDFs for each section are sorted in chronological semester order (August–December for 1st Semester; January–June for 2nd Semester).
   - Merged into `<OutputDir>/<Course_Sec>/Attendance/<Course_Sec>_<SchedCode>_ATTENDANCE_MERGED.pdf`.
2. **Academic Sequence CEIT Form Bundles**:
   - Academic forms are sorted according to official department order (`SYLLABUS_ACCEPTANCE`, `EXAM_RETURNS_MIDTERM`, `EXAM_RETURNS_FINALS`, `TOS_MIDTERM`, `TOS_FINALS`, `GRADE_DISCUSSION_MIDTERM`, `GRADE_DISCUSSION_FINALS`).
   - Merged into `<OutputDir>/<Course_Sec>/CEIT_Forms/<Course_Sec>_<SchedCode>_CEIT_FORMS_MERGED.pdf`.
3. **Artifact Tree Registration**:
   - Both individual PDFs and merged bundles are registered in `results["by_class"][course_sec]["pdf"]`.
   - The interactive Step 3 UI highlights merged bundles with a prominent `📑 Merged Bundle` badge and provides individual "Open" launch triggers.

---

## 🛑 Graceful Cancellation Engine

A core feature of the orchestrator is the ability to gracefully cancel a generation job mid-flight without corrupting data or leaving locked files.

- **Dual-Pass Cancellation**: Introduced in Commit `af78ff9`, the `process_all` function in `orchestrator.py` uses a dual-pass loop equipped with a `threading.Event` (`_cancel_event`).
- **State Polling**: During each generation phase, the orchestrator actively polls `_cancel_event.is_set()`. If a cancellation is requested via the PyWebView Bridge (`cancel_generation`), the loop breaks immediately.
- **Worker Clean Teardown**: `ConcurrentPdfPipeline` actively inspects `cancel_event` on every queued item, cleanly closes any active document, and uninitializes COM resources without leaving orphaned Word processes.
- **Cleanup**: Incomplete output directories or files generated during the cancelled pass are safely managed, ensuring no orphaned file locks prevent future generation attempts.

---

## 📡 JavaScript Evaluation Callbacks

The backend needs to update the UI thread on progress asynchronously.

- Instead of relying on return values, the orchestrator pushes progress updates using `window.evaluate_js()`.
- The `process_all` generator calculates total required steps dynamically (linear progress calculation based on detected subjects, selected engines, and optional concurrent PDF conversions + merges) and evaluates `window.onGenerationProgress(pct, msg)` to update the DOM progress bar and stepper UI in real-time.
