---
title: "HIG & UX Audit"
tags:
  - cvsu-generator
  - ui
  - ux
  - accessibility
  - design
status: active
last_modified: 2026-09-13
source_of_truth:
  - executable_test/ui.html
  - executable_test/css/
  - executable_test/js/
---

# HIG & UX Audit

This document provides a two-dimensional audit of the application's alignment with Apple Human Interface Guidelines (HIG), general UX best practices, and accessibility standards.

It evaluates the application by comparing historical design intent against the currently implemented state.

Related notes:
- [[UI Architecture]]
- [[Development Workflow]]

---

## 1. Visual Hierarchy & Theme

| Aspect | Historical Intent | Evidence in Code | Current State |
| :--- | :--- | :--- | :--- |
| **Minimalism & Spacing** | "Apple HIG minimalism" (Commit `544b238`) | `tokens.css` defines strict padding and gap tokens (`--spacing-md`, `--spacing-lg`). | **Present**. UI relies heavily on whitespace rather than borders to separate content blocks. |
| **Dark/Light Mode** | "Apple HIG velvet cross-dissolve" (Commit `830d54b`) | `ui.html` `<html data-theme="dark">` and `base.css` transition properties. | **Present**. Dynamic theme toggle fully implemented with CSS transitions mapping to token variables. |

## 2. Navigation

| Aspect | Historical Intent | Evidence in Code | Current State |
| :--- | :--- | :--- | :--- |
| **Workflow Stepper** | "Visual Stepper" (Commit `89cddfd`) | `stepper.js` and `ui.html` card IDs. | **Present**. Fixed 6-step horizontal progression clearly dividing the application state. |
| **Floating Dock** | "Floating dock HIG redesign" (Commit `6cfb685`) | `base.css` sticky positioning and z-index layers. | **Present**. Bottom navigation controls remain visible independent of scrolling content. |

## 3. Controls & Feedback

| Aspect | Historical Intent | Evidence in Code | Current State |
| :--- | :--- | :--- | :--- |
| **Confirmation Modals** | "Replace native browser confirm" (Commit `3ba8048`) | `modal.js` `showAppleConfirm` function triggering custom HTML dialog overlays. | **Present**. All destructive actions use the custom DOM modal instead of `window.confirm`. |
| **Non-blocking Notifications** | "Toast Notifications" (Commit `89cddfd`) | `app.js` `showToast` method rendering transient overlay messages. | **Present**. Non-critical feedback (e.g., file selected) does not block workflow. |
| **Generation Progress** | "Animated status indicators" (Commit `af78ff9`) | `process_all` yielding tokens to `window.onGenerationProgress` updating DOM widths. | **Present**. Real-time linear progress bar accurately reflects backend state. |

## 4. Accessibility (A11y)

| Aspect | Historical Intent | Evidence in Code | Current State |
| :--- | :--- | :--- | :--- |
| **Focus Traps** | "Comprehensive accessibility pass" (Commit `3022632`) | Keyboard event listeners inside `modal.js` intercepting the Tab key. | **Present**. Keyboard users cannot tab out of an active modal dialog. |
| **WCAG 2.1 AA Compliance** | "Axe violation policy tightening" (Commit `c2d467e`) | `tests/test_playwright_e2e.py` invoking `axe-core` analysis engine. | **Present**. Automated UI test suite fails on accessibility violations (e.g., contrast, ARIA labels). |

## 5. Platform Conventions

| Aspect | Historical Intent | Evidence in Code | Current State |
| :--- | :--- | :--- | :--- |
| **Native Drag and Drop** | "Native OLE AllowDrop" (Commit `c9e2c00`) | `dnd.py` `IDropTarget` interface intercepting Windows Explorer messages. | **Present**. File dragging bypasses the Chromium webview entirely and resolves paths at the OS level. |
| **Web Rendering Engine** | Single-file executable architecture (Commit `24944f4`) | PyWebView using WebView2 (Edge Chromium) under Windows. | **Present**. The application UI renders using standard web technologies but behaves as a native desktop client. |

---

## Final Verdict

The CvSU Document Generator exhibits strong adherence to its stated design intent. The interface successfully mimics Apple HIG aesthetic principles (minimalism, smooth transitions, custom modal patterns) despite running as a PyWebView application on Windows. 

Crucially, these aesthetic choices are underpinned by strict automated accessibility enforcement (axe-core) and OS-native integrations (OLE Drag and Drop), ensuring the application is both usable and compliant.
