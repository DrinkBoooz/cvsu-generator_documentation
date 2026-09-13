---
title: "Concurrency & Threading"
tags:
  - cvsu-generator
  - architecture
  - concurrency
  - threading
status: active
last_modified: 2026-09-13
source_of_truth:
  - modules/services/orchestrator.py
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
- Executed via PyWebView's background window execution or explicitly spawned threads.
- Handles I/O bound operations like parsing `.xlsx` rosters and generating `.docx`/`.xlsx` output files.

### 3. Native Windows Threading
- **Native OLE Drag and Drop**: Introduced in Commit `c9e2c00`, the application uses `pythonnet` to implement `IDropTarget` for native Windows file drag-and-drop. This avoids legacy Base64 JavaScript file drops and directly resolves paths at the OS level. Thread safety was overhauled to safely interface `IDropTarget` with the Python backend.

---

## 🛑 Graceful Cancellation Engine

A core feature of the orchestrator is the ability to gracefully cancel a generation job mid-flight without corrupting data or leaving locked files.

- **Dual-Pass Cancellation**: Introduced in Commit `af78ff9`, the `process_all` function in `orchestrator.py` uses a dual-pass loop equipped with a `threading.Event` (`_cancel_event`).
- **State Polling**: During each generation phase, the orchestrator actively polls `_cancel_event.is_set()`. If a cancellation is requested via the PyWebView Bridge (`cancel_generation`), the loop breaks immediately.
- **Cleanup**: Incomplete output directories or files generated during the cancelled pass are safely managed, ensuring no orphaned file locks prevent future generation attempts.

---

## 📡 JavaScript Evaluation Callbacks

The backend needs to update the UI thread on progress asynchronously.

- Instead of relying on return values, the orchestrator pushes progress updates using `window.evaluate_js()`.
- The `process_all` generator calculates total required steps dynamically (linear progress calculation based on detected subjects and selected engines) and evaluates `window.onGenerationProgress(pct, msg)` to update the DOM progress bar and stepper UI in real-time.
