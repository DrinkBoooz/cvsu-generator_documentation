---
title: "HIG & UX Audit"
tags:
  - cvsu-generator
  - ui
  - ux
  - accessibility
  - design
status: active
last_modified: 2026-09-14
source_of_truth:
  - executable_test/ui.html
  - executable_test/css/
  - executable_test/js/
---

# HIG & UX Audit

This document provides a comprehensive, evidence-backed evaluation of how the current **CvSU Document Generator** desktop user interface aligns with, partially aligns with, or intentionally diverges from Apple Human Interface Guidelines (HIG) principles and core cross-platform usability standards.

Related notes:
- [[CvSU Document Generator MOC]]
- [[UI Architecture]]
- [[Development Workflow]]
- [[User Manual]]

---

## 1. HIG Audit Scope & Evaluation Framework

The **CvSU Document Generator** is engineered as a desktop utility combining a Microsoft Edge WebView2 container (via PyWebView) with an HTML5/CSS/JavaScript single-page application shell and native Python/Windows OLE subsystem integration.

Because the software targets Windows desktop workstations while drawing stylistic and organizational inspiration from Apple's design philosophy (Commits `85`, `87`, `92`–`94`), this audit operates under an explicit **three-tiered evaluation model**:

1. **Tier 1: HIG-Relevant Cross-Platform Principles**: Universal usability principles emphasized in Apple's design philosophy that apply across desktop software regardless of operating system (clarity, visual hierarchy, feedback, error prevention and recovery, predictable form validation, direct manipulation, accessibility).
2. **Tier 2: Apple-Specific Platform Conventions**: Architectural paradigms designed exclusively for Apple operating systems (macOS global menu bar, macOS window-attached sheets, SF Symbols icon font, SF Pro native font rendering, iOS/iPadOS touch target minimums). These are evaluated neutrally; non-adoption on Windows is classified as **Not Applicable**.
3. **Tier 3: Windows-Native Conventions**: Desktop subsystem integrations tailored specifically for Windows environments (Windows OLE `IDropTarget` file drag-and-drop, standard single-window utility framing, `@media (forced-colors: active)` for Windows High Contrast). These are evaluated independently on their implementation fidelity and usability, receiving classifications such as **Platform-Appropriate / Aligned** or **Platform-Appropriate / Partially Aligned**.

---

## 2. Implementation & Empirical Evidence Baseline

This audit is derived strictly from current source code, rendered visual inspection, and automated runtime tests:

- **Application Markup & Layout**: `executable_test/ui.html` (3,166 lines; two-column grid layout, sticky stepper, bottom action bar, modal backdrops, and slide-over drawers).
- **Design Tokens & Stylesheets**:
  - `executable_test/css/tokens.css` (semantic color tokens, font sizing, spacing scales, easing curves).
  - `executable_test/css/base.css` (typography stack, focus rings, forced-colors rules).
  - `executable_test/css/components.css` (stepper chips, dropzones, badges, cards).
  - `executable_test/css/modals.css` (dialog backdrops, focus containment, animation keyframes).
  - `executable_test/css/drawers.css` (slide-over aside panels, dock action bar, toast styles).
  - `executable_test/css/tables.css` (table layouts, sticky headers, loading animation).
- **Interaction Controllers**: Modular ES6 controllers under `executable_test/js/` (`state.js`, `toast.js`, `modal.js`, `theme.js`, `drawers.js`, `stepper.js`, `bridge.js`, `step1.js`, `step2.js`, `step3.js`, `settings.js`, `templates.js`, `app.js`).
- **Native Subsystem**: `executable_test/native/dnd.py` (Windows Forms / WebView2 `AllowDrop` integration and `DOMEventHandler`).
- **Runtime Automated Verification**: 19 passing Playwright end-to-end and accessibility tests (`test_playwright_e2e.py`, `test_playwright_accessibility.py`, `test_playwright_roster_mapping.py`, `test_playwright_settings_modal.py`).
- **Visual Inspection Artifacts**: Headless Chromium captures (1200×800 viewport; `initial_ui.png`, `settings_modal.png`, `help_drawer.png`) and computed style extractions (`visual_metrics.json`).

---

## 3. Official Apple HIG Guidance Sources Referenced

Every substantive assessment references current official Apple Developer documentation:

1. **Layout & Visual Hierarchy**: *Layout | Apple Developer Documentation* ([developer.apple.com/design/human-interface-guidelines/layout](https://developer.apple.com/design/human-interface-guidelines/layout))
   - *Relevant Apple guidance*: Use negative space, size, and visual weight to establish a clear hierarchy of importance that directs attention to essential tasks.
2. **Typography**: *Typography | Apple Developer Documentation* ([developer.apple.com/design/human-interface-guidelines/typography](https://developer.apple.com/design/human-interface-guidelines/typography))
   - *Relevant Apple guidance*: Text must remain legible across display scales; interfaces should maintain optical balance and clear hierarchy regardless of host platform typeface.
3. **Navigation**: *Navigation and search | Apple Developer Documentation* ([developer.apple.com/design/human-interface-guidelines/navigation-and-search](https://developer.apple.com/design/human-interface-guidelines/navigation-and-search))
   - *Relevant Apple guidance*: Design clear pathways through the app so users always know their current location, completed steps, and upcoming requirements.
4. **Drag and Drop**: *Drag and drop | Apple Developer Documentation* ([developer.apple.com/design/human-interface-guidelines/drag-and-drop](https://developer.apple.com/design/human-interface-guidelines/drag-and-drop))
   - *Relevant Apple guidance*: Provide direct manipulation with immediate, unmistakable visual feedback confirming drop target affordance and state.
5. **Buttons & Controls**: *Buttons | Apple Developer Documentation* ([developer.apple.com/design/human-interface-guidelines/buttons](https://developer.apple.com/design/human-interface-guidelines/buttons))
   - *Relevant Apple guidance*: Clearly communicate control purpose, interactive affordance, and state changes, providing adequate target areas for pointer acquisition.
6. **Entering Data**: *Entering data | Apple Developer Documentation* ([developer.apple.com/design/human-interface-guidelines/entering-data](https://developer.apple.com/design/human-interface-guidelines/entering-data))
   - *Relevant Apple guidance*: Validate user input predictably and provide timely guidance when errors occur, ideally while the user is actively working.
7. **Feedback & Progress Indicators**: *Feedback | Apple Developer Documentation* ([developer.apple.com/design/human-interface-guidelines/feedback](https://developer.apple.com/design/human-interface-guidelines/feedback)) & *Progress indicators | Apple Developer Documentation* ([developer.apple.com/design/human-interface-guidelines/progress-indicators](https://developer.apple.com/design/human-interface-guidelines/progress-indicators))
   - *Relevant Apple guidance*: Offer timely, non-intrusive status updates and quantifiable progress for operations that take more than a few seconds.
8. **Alerts & Destructive Actions**: *Alerts | Apple Developer Documentation* ([developer.apple.com/design/human-interface-guidelines/alerts](https://developer.apple.com/design/human-interface-guidelines/alerts))
   - *Relevant Apple guidance*: Reserve modal alerts for critical or irreversible actions. Use action-oriented button labels and always include a safe Cancel option.
9. **Modality**: *Modality | Apple Developer Documentation* ([developer.apple.com/design/human-interface-guidelines/modality](https://developer.apple.com/design/human-interface-guidelines/modality)) & *Sheets | Apple Developer Documentation* ([developer.apple.com/design/human-interface-guidelines/sheets](https://developer.apple.com/design/human-interface-guidelines/sheets))
   - *Relevant Apple guidance*: Use modality to present content in a dedicated mode that prevents interaction with the parent view and requires an explicit action to dismiss, reserving it for critical subtasks.
10. **Motion & Accessibility**: *Motion | Apple Developer Documentation* ([developer.apple.com/design/human-interface-guidelines/motion](https://developer.apple.com/design/human-interface-guidelines/motion)) & *Accessibility | Apple Developer Documentation* ([developer.apple.com/design/human-interface-guidelines/accessibility](https://developer.apple.com/design/human-interface-guidelines/accessibility))
    - *Relevant Apple guidance*: Keep animations brief and purposeful; provide alternatives or reduced motion for users sensitive to animated transitions.
11. **Color & Appearances**: *Color | Apple Developer Documentation* ([developer.apple.com/design/human-interface-guidelines/color](https://developer.apple.com/design/human-interface-guidelines/color)) & *Dark Mode | Apple Developer Documentation* ([developer.apple.com/design/human-interface-guidelines/dark-mode](https://developer.apple.com/design/human-interface-guidelines/dark-mode))
    - *Relevant Apple guidance*: Use color to support hierarchy and state, but never as the sole indicator of essential information; adapt seamlessly between light and dark appearances.
12. **Windows**: *Windows | Apple Developer Documentation* ([developer.apple.com/design/human-interface-guidelines/windows](https://developer.apple.com/design/human-interface-guidelines/windows))
    - *Relevant Apple guidance*: Organize window frames and content to fit the application's purpose and the host operating system's conventions.
13. **The Menu Bar**: *The menu bar | Apple Developer Documentation* ([developer.apple.com/design/human-interface-guidelines/the-menu-bar](https://developer.apple.com/design/human-interface-guidelines/the-menu-bar))
    - *Relevant Apple guidance*: In macOS, the system menu bar at the top of the screen provides persistent access to application commands.

---

## 4. Final HIG Assessment Matrix

| HIG Area | Current Implementation | Official Apple Source | Applicability | Classification | Evidence | Confidence |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Visual Hierarchy** | Two-column dashboard grid; prominent `#bottomActionBar` with primary trigger `#btnDockProcess`; glass-card containers. | *Layout* ([Apple HIG](https://developer.apple.com/design/human-interface-guidelines/layout)) | HIG-relevant cross-platform principle | **Aligned** | Source (`ui.html`) + Visual (1200×800 capture) | High |
| **Typography** | Font stack: `-apple-system, BlinkMacSystemFont, "SF Pro Text", "SF Pro Display", "Segoe UI", Roboto, Arial, sans-serif`. Resolves to `Segoe UI` on Windows. | *Typography* ([Apple HIG](https://developer.apple.com/design/human-interface-guidelines/typography)) | HIG-relevant cross-platform principle / Apple-inspired | **Platform-Appropriate / Aligned** | Source (`tokens.css`, `base.css`) + Visual (computed styles) | High |
| **Navigation & Stepper** | Sticky 6-step progress bar (`#workflowStepper`) with step chips (`#chipStep1` to `#chipStep6`), dynamic status icons, and viewport scroll-spy. | *Navigation and search* ([Apple HIG](https://developer.apple.com/design/human-interface-guidelines/navigation-and-search)) | HIG-relevant cross-platform principle | **Aligned** | Source (`stepper.js`) + Runtime (`test_playwright_e2e.py`) + Visual | High |
| **Direct Manipulation & Drag/Drop** | Windows native OLE `IDropTarget` (`dnd.py`) routing through `bridge.js`; visual dragover state on `.dropzone`; keyboard file picker alternative. | *Drag and drop* ([Apple HIG](https://developer.apple.com/design/human-interface-guidelines/drag-and-drop)) | Windows-native convention | **Platform-Appropriate / Aligned** | Source (`dnd.py`, `bridge.js`) + Runtime (`test_playwright_e2e.py`) | High |
| **Desktop Controls & Target Sizing** | Primary button (`padding: 10px 22px`), 36×36px utility icons, segmented controls (`.segmented-btn`), focus rings (`--a11y-focus-ring-width: 2px`). | *Buttons* ([Apple HIG](https://developer.apple.com/design/human-interface-guidelines/buttons)) | HIG-relevant cross-platform principle | **Aligned** | Source (`drawers.css`, `components.css`) + Visual | High |
| **Form Inputs & Validation** | Date range inputs with chronology check; pre-flight validation on generation trigger with error toasts; limited real-time inline field validation. | *Entering data* ([Apple HIG](https://developer.apple.com/design/human-interface-guidelines/entering-data)) | HIG-relevant cross-platform principle | **Partially Aligned** | Source (`step2.js`, `step3.js`) + Runtime (`test_playwright_e2e.py`) | High |
| **Feedback & Telemetry** | Non-modal toast system (`#toastContainer`, `js/toast.js`); WAI-ARIA `progressbar` pattern; live elapsed stopwatch timer. | *Feedback* & *Progress indicators* ([Apple HIG](https://developer.apple.com/design/human-interface-guidelines/feedback)) | HIG-relevant cross-platform principle | **Aligned** | Source (`toast.js`, `ui.html`) + Runtime (`test_playwright_e2e.py`) | High |
| **Errors & Recovery** | Graceful rejection of invalid files via warning/error toasts; generation double-click guards; responsive thread abort via `#btnCancelGeneration`. | *Alerts* & *Feedback* ([Apple HIG](https://developer.apple.com/design/human-interface-guidelines/alerts)) | HIG-relevant cross-platform principle | **Aligned** | Source (`step3.js`) + Runtime (`test_playwright_e2e.py`) | High |
| **Destructive Action Safeguards** | Centered confirmation modal (`#modalAppleConfirmBackdrop`) invoked for irreversible configuration resets and file replacements. Clear button titles, safe Cancel, Escape dismissal, focus return. | *Alerts* ([Apple HIG](https://developer.apple.com/design/human-interface-guidelines/alerts)) | HIG-relevant cross-platform principle | **Aligned** | Source (`modal.js`, `settings.js`) + Runtime (`test_playwright_accessibility.py`) + Visual | High |
| **Modal Presentation & Modality** | Centered window backdrop overlays (`#modalParserSettingsBackdrop`, `#modalRosterMappingBackdrop`, `#modalAppleConfirmBackdrop`) with focus trapping and Escape key dismissal. | *Modality* ([Apple HIG](https://developer.apple.com/design/human-interface-guidelines/modality)) & *Sheets* ([Apple HIG](https://developer.apple.com/design/human-interface-guidelines/sheets)) | Windows-native / Webview context | **Platform-Appropriate / Aligned** | Source (`modal.js`, `modals.css`) + Runtime + Visual | High |
| **Motion & Animation** | CSS transitions (150ms–300ms) with spring-inspired cubic-bezier curves; modal pop (8px/4% scale); drawer slide (100% horizontal); absence of `@media (prefers-reduced-motion)`. | *Motion* & *Accessibility* ([Apple HIG](https://developer.apple.com/design/human-interface-guidelines/motion)) | HIG-relevant cross-platform principle / Accessibility | **Partially Aligned** | Source (`tokens.css`, `modals.css`, `drawers.css`) + Visual | High |
| **Color & Visual States** | Tailored HSL palette; light and dark modes; non-color indicators (icons, badges, text labels, borders, opacity) across tested workflow states; `@media (forced-colors: active)` in CSS. | *Color* & *Dark Mode* ([Apple HIG](https://developer.apple.com/design/human-interface-guidelines/color)) | HIG-relevant cross-platform principle | **Aligned** | Source (`tokens.css`, `base.css`, `components.css`) + Visual | High |
| **Window Anatomy & Chrome** | Single-window utility layout with top header; absence of traditional desktop menu bar or keyboard shortcut accelerators for drawer access. | *Windows* ([Apple HIG](https://developer.apple.com/design/human-interface-guidelines/windows)) | Windows-native / Webview context | **Platform-Appropriate / Partially Aligned** | Source (`ui.html`) + Visual | Medium |
| **macOS System Menu Bar** | No global top menu bar. | *The menu bar* ([Apple HIG](https://developer.apple.com/design/human-interface-guidelines/the-menu-bar)) | Apple-specific platform convention | **Not Applicable** | Platform Architecture (Windows WebView2 runtime) | High |

---

## 5. Detailed HIG Area Audits

### 5.1 Visual Hierarchy & Layout
- **Current Implementation**: The main workspace (`ui.html`) employs a structured two-column layout. The left column organizes file ingestion (Step 1 Master Schedule and Step 2 Class Rosters); the right column organizes generator parameters (Step 3 Engines, Step 4 Dates, Step 5 Output Folder). Primary execution is anchored persistently in `#bottomActionBar` with `#btnDockProcess`.
- **Official Apple Source**: *Layout | Apple Developer Documentation* ([developer.apple.com/design/human-interface-guidelines/layout](https://developer.apple.com/design/human-interface-guidelines/layout)).
- **Relevant Principle**: Apple guidance emphasizes using negative space, size, and visual weight to establish an unmistakable hierarchy of importance, keeping primary actions prominent and secondary controls subordinate.
- **Applicability**: HIG-relevant cross-platform principle.
- **Assessment**: **Aligned**. The visual hierarchy clearly differentiates primary workflow cards from global utilities. Secondary tools (Theme, Settings, Logs, Help) reside in the slim top navigation header, preventing visual clutter.
- **Evidence & Confidence**: Visual inspection (1200×800 capture) and DOM structure (`ui.html:56–1212`). High confidence.

### 5.2 Typography & Optical Balance
- **Current Implementation**: Body typography specifies a prioritized system font stack: `-apple-system, BlinkMacSystemFont, "SF Pro Text", "SF Pro Display", "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif` (`tokens.css:4`).
- **Official Apple Source**: *Typography | Apple Developer Documentation* ([developer.apple.com/design/human-interface-guidelines/typography](https://developer.apple.com/design/human-interface-guidelines/typography)).
- **Relevant Principle**: Apple guidance emphasizes that text must remain legible at every size, with consistent optical balance and proportional hierarchy across platforms.
- **Applicability**: HIG-relevant cross-platform principle / Apple-inspired convention.
- **Assessment**: **Platform-Appropriate / Aligned**.
  - Font fallback itself is not treated as evidence of HIG compliance; the assessment is grounded in optical readability, hierarchy contrast, and observed rendering in the Windows runtime environment.
  - On standard Windows workstations without Apple proprietary fonts installed, the browser engine falls back cleanly to `Segoe UI` with crisp subpixel rendering. Optical balance is maintained through standardized scale variables (`--hig-font-body: 15px`, `--hig-font-title-1: 28px`, `--hig-font-caption-1: 12px`).
  - SF Pro is not assumed to be actively rendered unless confirmed installed on the host environment.
- **Evidence & Confidence**: Verified via computed styles in headless Chromium (`bodyFont` resolves to Segoe UI; `visual_metrics.json`). High confidence.

### 5.3 Navigation & Workflow Stepper
- **Current Implementation**: A sticky workflow stepper (`#workflowStepper`) provides 6 interactive step chips (`#chipStep1` to `#chipStep6`) with animated connectors (`#connector1to2`, etc.). Chips update dynamically with icons (clock, checkmark, warning) and status labels ("Incomplete", "Complete"). Clicking a chip smoothly scrolls to the target card.
- **Official Apple Source**: *Navigation and search | Apple Developer Documentation* ([developer.apple.com/design/human-interface-guidelines/navigation-and-search](https://developer.apple.com/design/human-interface-guidelines/navigation-and-search)).
- **Relevant Principle**: Apple guidance emphasizes intuitive pathways that keep users aware of their current state, past steps, and remaining tasks without trapping them in rigid modal flows.
- **Applicability**: HIG-relevant cross-platform principle.
- **Assessment**: **Aligned**. The linear milestone progression provides clear spatial orientation and direct navigation without trapping users in modal wizards.
- **Evidence & Confidence**: Verified in runtime tests (`test_playwright_e2e.py`) and source (`js/stepper.js`). High confidence.

### 5.4 Direct Manipulation & Native Windows Drag-and-Drop
- **Current Implementation**: The application uses the native Windows OLE `IDropTarget` integration implemented in `executable_test/native/dnd.py`, with JavaScript routing handled through `executable_test/js/bridge.js`.
- **Official Apple Source**: *Drag and drop | Apple Developer Documentation* ([developer.apple.com/design/human-interface-guidelines/drag-and-drop](https://developer.apple.com/design/human-interface-guidelines/drag-and-drop)).
- **Relevant Principle**: Apple guidance highlights drag-and-drop as an intuitive direct manipulation technique that requires distinct visual feedback confirming drop target readiness and data acceptance.
- **Applicability**: Windows-native convention.
- **Assessment**: **Platform-Appropriate / Aligned**.
  - *Discoverability*: Dropzones (`#scheduleDropzone`, `#rostersDropzone`) are prominently positioned within Step 1 and Step 2 cards with dashed borders (`border: 2px dashed`), upload glyphs, and clear instructions ("Drop file here or click to browse").
  - *Dropzone Affordance*: Distinct border styling, background contrast, and cursor styling visually communicate drop acceptance.
  - *Drag-Over Feedback*: `js/bridge.js` applies `.drag-over` / `.active` classes on `dragenter` and `dragover`, updating border illumination to brand accent (`--accent-emerald`).
  - *Acceptance Feedback*: Dropping a file immediately displays a file badge, file size, checkmark icon, and initiates parsing status.
  - *Error Handling*: Dropping unsupported file formats triggers warning toasts without stalling or crashing the application shell.
  - *Keyboard Alternative*: Dropzones declare `tabindex="0"`, `role="button"`, and invoke the native file picker when Enter or Space is pressed.
- **Evidence & Confidence**: Source (`dnd.py`, `bridge.js`) and runtime E2E test execution. High confidence.

### 5.5 Desktop Controls & Pointer Target Sizing
- **Current Implementation**: Interactive controls provide distinct hover, active, and focus states. Primary trigger (`#btnDockProcess`) has measured padding of `10px 22px` and font-size `13.5px` (rendered bounds ~40px height, ~180px width). Top utility buttons (`#btnToggleTheme`, `#btnOpenSettings`, `#btnOpenHelp`) measure `36×36px` with 8px internal padding.
- **Official Apple Source**: *Buttons | Apple Developer Documentation* ([developer.apple.com/design/human-interface-guidelines/buttons](https://developer.apple.com/design/human-interface-guidelines/buttons)).
- **Relevant Principle**: Apple guidance emphasizes that controls must clearly communicate their purpose and state, offering adequately sized targets for reliable pointer acquisition.
- **Applicability**: HIG-relevant cross-platform principle.
- **Assessment**: **Aligned**. Primary controls provide sufficiently large and clearly defined pointer targets for the current desktop interface. Visual feedback includes active press scaling (`--hig-press-scale: 0.98`, `transform: scale(0.97)`), elevation shadows, and high-visibility focus rings (`outline: 2px solid var(--accent-emerald)`). Mobile touch-target standards (e.g. 44×44pt) are not treated as binding requirements for this desktop application.
- **Evidence & Confidence**: Source (`drawers.css:462–505`, `components.css`) and visual inspection. High confidence.

### 5.6 Form Inputs & Validation Timing
- **Current Implementation**: Date inputs enforce chronological boundaries (`endDate >= startDate`). Multi-step configuration errors across Steps 1–5 are validated predominantly during pre-flight checks when `#btnDockProcess` is clicked in `js/step3.js`, emitting error toasts and scrolling the viewport to the offending section.
- **Official Apple Source**: *Entering data | Apple Developer Documentation* ([developer.apple.com/design/human-interface-guidelines/entering-data](https://developer.apple.com/design/human-interface-guidelines/entering-data)).
- **Relevant Principle**: Apple guidance recommends validating input as soon as possible, ideally inline while the user is actively working, rather than deferring all feedback to submission time.
- **Applicability**: HIG-relevant cross-platform principle.
- **Assessment**: **Partially Aligned**. Pre-flight validation reliably guards the generation engine against corrupted inputs, but the absence of real-time inline field validation banners or dirty-checking prior to clicking the primary action means users may experience friction when submitting incomplete setups.
- **Evidence & Confidence**: Source (`step3.js:45–95`) and runtime validation failure tests. High confidence.

### 5.7 Feedback, Live Telemetry & Progress
- **Current Implementation**: Long-running generation jobs trigger `#progressContainer`, which implements the WAI-ARIA Progressbar pattern (`role="progressbar"`, `aria-valuemin="0"`, `aria-valuemax="100"`, dynamic `aria-valuenow`, dynamic `aria-valuetext`). A live stopwatch timer (`#progressElapsedTimer`) displays elapsed execution time. Discrete events and non-blocking warnings are dispatched via non-modal toasts (`#toastContainer`, `js/toast.js`).
- **Official Apple Source**: *Feedback | Apple Developer Documentation* ([developer.apple.com/design/human-interface-guidelines/feedback](https://developer.apple.com/design/human-interface-guidelines/feedback)) & *Progress indicators | Apple Developer Documentation* ([developer.apple.com/design/human-interface-guidelines/progress-indicators](https://developer.apple.com/design/human-interface-guidelines/progress-indicators)).
- **Relevant Principle**: Apple guidance advises providing non-intrusive status updates and quantifiable progress indicators for long-running operations without blocking user interaction unnecessarily.
- **Applicability**: HIG-relevant cross-platform principle.
- **Assessment**: **Aligned**. Real-time progress updates, stopwatch telemetry, and transient live notifications communicate system state clearly without interrupting workflow execution.
- **Evidence & Confidence**: Source (`ui.html:1130–1165`, `toast.js`) and runtime test assertions. High confidence.

### 5.8 Destructive Action Safeguards & Confirmation
- **Current Implementation**: The application provides a reusable modal confirmation engine (`showAppleConfirm` in `js/modal.js`).
  - *Actions Invoking Confirmation*:
    1. Factory configuration reset (`settings.js:523`): Resets user-defined keyword tokens, degree alias mappings, and course prefixes to default values.
    2. Ingested schedule replacement (`step1.js:145`): Overwrites an existing loaded schedule workbook and resets parsed class metadata.
    3. Custom template slot override/removal (`templates.js:240`).
  - *Destructive & Reversibility Assessment*: These actions permanently overwrite active session data and `localStorage` configurations. They cannot be undone without manual re-entry.
  - *Confirm Action*: Explicitly labeled button ("Reset Defaults", "Replace") styled in destructive rose red (`--accent-rose`), executes the reset, and dismisses the dialog.
  - *Cancel Action*: Distinct neutral "Cancel" button, safely aborts without state changes, and dismisses the dialog.
  - *Keyboard & Focus*: Escape key safely cancels and dismisses (`dismissAppleConfirm()`). Focus returns cleanly to the originating trigger button.
- **Official Apple Source**: *Alerts | Apple Developer Documentation* ([developer.apple.com/design/human-interface-guidelines/alerts](https://developer.apple.com/design/human-interface-guidelines/alerts)).
- **Relevant Principle**: Apple guidance advises using alerts only for genuinely destructive or irreversible actions. Buttons should use action-oriented titles rather than vague acknowledgments, and a safe Cancel option must always be included.
- **Applicability**: HIG-relevant cross-platform principle.
- **Assessment**: **Aligned**. The confirmation modal satisfies criteria for necessity, clarity, proportionality, reversibility warning, and safe cancellation. Routine actions never prompt alerts.
- **Evidence & Confidence**: Source (`modal.js:142–200`, `settings.js:523`) and runtime accessibility test (`test_playwright_accessibility.py`). High confidence.

### 5.9 Modal Presentation & Modality
- **Current Implementation**: The current Windows/PyWebView implementation uses centered backdrop overlays with focus trapping, Escape dismissal, and focus restoration (`#modalParserSettingsBackdrop`, `#modalRosterMappingBackdrop`, `#modalAppleConfirmBackdrop`). macOS-specific window-attached sheet behavior is not directly applicable to this Windows implementation.
- **Official Apple Source**: *Modality | Apple Developer Documentation* ([developer.apple.com/design/human-interface-guidelines/modality](https://developer.apple.com/design/human-interface-guidelines/modality)) & *Sheets | Apple Developer Documentation* ([developer.apple.com/design/human-interface-guidelines/sheets](https://developer.apple.com/design/human-interface-guidelines/sheets)).
- **Relevant Principle**: Apple guidance defines modality as a technique that presents content in a separate, dedicated mode that prevents interaction with the parent view and requires an explicit action to dismiss, reserving it for critical subtasks that require deliberate user choices.
- **Applicability**: Windows-native / Webview context.
- **Assessment**: **Platform-Appropriate / Aligned**. Centered backdrop overlays represent the appropriate idiom for Windows desktop webviews. Modals enforce strict focus trapping (Tab/Shift+Tab boundary cycling), dismiss on Escape, and restore focus to trigger buttons upon dismissal.
- **Evidence & Confidence**: Source (`modal.js`, `modals.css`) and runtime focus trapping tests. High confidence.

### 5.10 Motion, Animation & Transitions
- **Current Implementation**:
  - CSS transitions: `0.15s–0.3s cubic-bezier(0.16, 1, 0.3, 1)` on cards, buttons, and chips.
  - Modal appearance: `modalPop` keyframe (`scale(0.96) translateY(8px) -> scale(1) translateY(0)` over `0.25s`).
  - Drawers: Horizontal slide-over `translateX(100%) -> translateX(0)` over `0.3s`.
  - Toast notifications: `toastSlideIn` (`translateY(16px) -> translateY(0)` over `0.25s`).
  - Theme toggle: CSS variable cross-dissolve (300ms) with optional iris reveal `appleThemeIrisReveal` (`0.65s`).
  - Absence of Media Query: No `@media (prefers-reduced-motion)` exists in `executable_test/css/`.
- **Official Apple Source**: *Motion | Apple Developer Documentation* ([developer.apple.com/design/human-interface-guidelines/motion](https://developer.apple.com/design/human-interface-guidelines/motion)) & *Accessibility | Apple Developer Documentation* ([developer.apple.com/design/human-interface-guidelines/accessibility](https://developer.apple.com/design/human-interface-guidelines/accessibility)).
- **Relevant Principle**: Apple guidance recommends keeping animations brief and purposeful to avoid delay or disorientation, and providing alternatives when people enable system-level reduced-motion preferences.
- **Applicability**: HIG-relevant cross-platform principle / Accessibility.
- **Assessment**: **Partially Aligned**.
  - *Duration & Amplitude*: Animations are brief (150ms–300ms), localized, and low-amplitude (e.g. 8px vertical shift on modals, 16px on toasts). They do not involve full-screen 3D flips, continuous parallax, or disorienting camera movements.
  - *Purpose & Frequency*: Motion communicates spatial hierarchy (drawers slide from the viewport edge; toasts emerge from bottom right) and is triggered strictly by user action.
  - *Limitation*: Because the application does not query OS-level reduced motion preferences, users who request reduced animation through Windows accessibility settings cannot suppress transitions.
- **Evidence & Confidence**: Source inspection of all stylesheets under `executable_test/css/`. High confidence.

### 5.11 Color, Contrast & State Independence
- **Current Implementation**:
  - Color Tokens: Tailored HSL palette featuring Emerald green (`--accent-emerald: #059669` light, `#10b981` dark), Amber (`--accent-amber: #d97706` light, `#f59e0b` dark), Rose red (`--accent-rose: #dc2626` light, `#ef4444` dark), and Slate base (`--bg-base: #f8fafc` light, `#0b0f17` dark).
  - State Cue Inspection:
    - *Incomplete*: Gray border, clock glyph, explicit text badge ("Incomplete").
    - *Complete*: Emerald highlight, checkmark glyph, explicit text badge ("Complete").
    - *Success*: Green border, checkmark icon, bold "Success" title text.
    - *Warning*: Amber border, triangle exclamation icon, bold "Warning" title text.
    - *Error*: Red border, circle X icon, bold "Error" title text.
    - *Disabled*: Explicit opacity (`0.55`), `cursor: not-allowed`, and `filter: grayscale(0.5)`.
    - *Active / Focus*: Pill border highlight, active scale, and 2px high-visibility focus ring (`outline: 2px solid var(--accent-emerald); outline-offset: 2px`).
- **Official Apple Source**: *Color | Apple Developer Documentation* ([developer.apple.com/design/human-interface-guidelines/color](https://developer.apple.com/design/human-interface-guidelines/color)) & *Dark Mode | Apple Developer Documentation* ([developer.apple.com/design/human-interface-guidelines/dark-mode](https://developer.apple.com/design/human-interface-guidelines/dark-mode)).
- **Relevant Principle**: Apple guidance emphasizes using color to support visual hierarchy and clarify state, while never relying on color as the sole means of conveying critical information.
- **Applicability**: HIG-relevant cross-platform principle.
- **Assessment**: **Aligned**. Across all audited workflow states (stepper progression, toasts, dock status pills, buttons, and focus rings), color changes are paired with non-color indicators (icons, text labels, opacity, cursor styles, or geometric outlines). Theme switching smoothly cross-dissolves token values.
- **Evidence & Confidence**: Source (`tokens.css`, `base.css`, `drawers.css`) and visual inspection. High confidence.

### 5.12 Window Anatomy & Desktop Chrome
- **Current Implementation**: Single-window utility shell without a traditional desktop menu bar (File, Edit, View, Help). Auxiliary functions are accessed via header icon buttons that trigger slide-over drawers (`#helpDrawer`, `#logsDrawer`) or modal dialogs (`#modalParserSettingsBackdrop`).
- **Official Apple Source**: *Windows | Apple Developer Documentation* ([developer.apple.com/design/human-interface-guidelines/windows](https://developer.apple.com/design/human-interface-guidelines/windows)).
- **Relevant Principle**: Apple guidance advises structuring window anatomy to fit application content and platform conventions, ensuring essential commands remain easily accessible.
- **Applicability**: Windows-native / Webview context.
- **Assessment**: **Platform-Appropriate / Partially Aligned**. The single-window layout is platform-appropriate for a dedicated document generation utility. However, the absence of global keyboard shortcut accelerators (e.g. `Ctrl+,` for Settings, `Ctrl+L` for Logs, `F1` for Help) leaves drawer discovery reliant solely on mouse pointer interaction.
- **Evidence & Confidence**: Source (`ui.html:36–54`) and visual layout inspection. Medium confidence.

### 5.13 macOS Global System Menu Bar
- **Current Implementation**: No global system menu bar exists.
- **Official Apple Source**: *The menu bar | Apple Developer Documentation* ([developer.apple.com/design/human-interface-guidelines/the-menu-bar](https://developer.apple.com/design/human-interface-guidelines/the-menu-bar)).
- **Relevant Principle**: In macOS, the menu bar is always available at the top of the screen and contains menus for app-level commands.
- **Applicability**: Apple-specific platform convention.
- **Assessment**: **Not Applicable**. The application targets Windows desktop environments where global macOS menu bars do not exist and are not expected by users.
- **Evidence & Confidence**: Target runtime platform architecture (Windows WebView2). High confidence.

---

## 6. Significant HIG Deviations (Evidence-Supported)

### Deviation 1: Absence of Reduced-Motion Query Handling
- **HIG Area**: Motion & Accessibility (*Apple HIG — Motion*, *Apple HIG — Accessibility*).
- **Current Implementation**: Transitions (150ms–300ms) and keyframe animations (`modalPop`, `toastSlideIn`, `pulse-emerald`) execute unconditionally without `@media (prefers-reduced-motion)`.
- **Why It Differs**: The stylesheet architecture prioritized self-contained micro-animations (`--hig-ease-spring`) without wiring system-level accessibility media query listeners.
- **Empirical Impact**: Low to Medium. Animations are brief, purposeful, and localized to small containers; they avoid full-screen disorientation. However, users who request reduced animation through Windows accessibility settings cannot suppress UI transitions.
- **Confidence**: **High** (Verified by exhaustive search across all files in `executable_test/css/`).

### Deviation 2: Pre-Flight Only Form Validation
- **HIG Area**: Entering Data (*Apple HIG — Entering data*).
- **Current Implementation**: Multi-step configuration errors (e.g. unselected generation engines or unconfigured date bounds) are validated predominantly during pre-flight checks when the primary action button (`#btnDockProcess`) is clicked, triggering toasts and scrolling to the error section.
- **Why It Differs**: Decouples individual step logic from complex multi-field dirty checking, avoiding premature warnings before the user has finished interacting.
- **Empirical Impact**: Low. The stepper chips display status badges, but users may click "Initialize Workflow" without realizing a required field was skipped.
- **Confidence**: **High** (Verified in `js/step3.js:45–95` and Playwright validation tests).

---

## 7. Platform-Appropriate Differences

### 7.1 Windows OLE Drag-and-Drop Integration
- **Behavior**: The application uses the native Windows OLE `IDropTarget` integration implemented in `executable_test/native/dnd.py`, with JavaScript routing handled through `executable_test/js/bridge.js`.
- **Comparison to Apple Conventions**: macOS WebKit applications typically route drag-and-drop through native Cocoa delegates. Under Windows Edge WebView2, standard HTML5 drag-and-drop can be subject to sandboxing or security boundaries.
- **Platform Rationale**: OLE integration provides zero file-path truncation across local and networked Windows drives, delivering the direct file drop behavior expected by Windows desktop users.
- **Usability Evaluation**: **Platform-Appropriate / Aligned**. Preserves direct manipulation, visible dragover illumination, and clear dropzone feedback.

### 7.2 Windows High Contrast Mode (`forced-colors: active`)
- **Behavior**: `executable_test/css/base.css` (lines 61–108) defines explicit rules under `@media (forced-colors: active)`, assigning high-visibility system colors (`Highlight`, `CanvasText`, `ButtonBorder`) to cards, inputs, dropzones, and active stepper chips.
- **Comparison to Apple Conventions**: Apple platforms utilize system-level accessibility filters and dynamic semantic colors without a direct equivalent to CSS forced-colors mode.
- **Platform Rationale**: Honors Windows Accessibility contrast themes (Desert, Aquatic, Night Sky, Dusk) directly inside the WebView2 container.
- **Usability Evaluation**: **Platform-Appropriate / Aligned** (Source verified).
  - *Verification Boundary*: Source rules are verified in `base.css`. Live visual rendering under an active Windows High Contrast session was not executed in this headless test environment.

### 7.3 Persistent Floating Bottom Action Bar (`#bottomActionBar`)
- **Behavior**: The primary execution trigger (`#btnDockProcess`) and readiness pills (`#actionPillSchedule`, `#actionPillRosters`, `#actionPillOutput`) are housed in a fixed bottom dock (`bottom: 14px`, `position: fixed`).
- **Comparison to Apple Conventions**: macOS desktop applications typically place primary toolbar actions at the top of the window chrome or inside modal sheets.
- **Platform Rationale**: In a vertical scrolling layout with extensive parameter cards, anchoring the primary trigger and readiness summary at the bottom right ensures constant visibility and discoverability without requiring continuous scrolling.
- **Usability Evaluation**: **Platform-Appropriate / Aligned**. Maintains clear visual hierarchy and instant action discovery.

### 7.4 Centered Modal Overlays vs macOS Sheets
- **Behavior**: The current Windows/PyWebView implementation uses centered backdrop overlays with focus trapping, Escape dismissal, and focus restoration (`#modalParserSettingsBackdrop`, `#modalAppleConfirmBackdrop`). macOS-specific window-attached sheet behavior is not directly applicable to this Windows implementation.
- **Comparison to Apple Conventions**: macOS applications frequently present document-centric transient tasks as window-attached modal sheets sliding from beneath the title bar.
- **Platform Rationale**: Centered overlays represent the natural idiom for Windows desktop web applications, avoiding artificial emulations of macOS sheet mechanics.
- **Usability Evaluation**: **Platform-Appropriate / Aligned**. Implements robust focus containment, backdrop dimming, and Escape key dismissal.

---

## 8. Accessibility Implementation & Forensic Boundaries

The accessibility audit observes strict empirical verification boundaries:

1. **Focus Containment & Navigation**:
   - *Runtime Verified*: `test_playwright_accessibility.py` confirmed that `#modalParserSettingsBackdrop` and `#modalAppleConfirmBackdrop` trap Tab/Shift+Tab focus within boundary elements, dismiss on Escape, and restore focus to the originating trigger button (`#btnOpenSettings` / `#btnResetDefaults`).
2. **Keyboard Activation**:
   - *Runtime Verified*: Dropzones (`#scheduleDropzone`, `#rostersDropzone`) declare `tabindex="0"`, `role="button"`, and trigger file selection via Enter and Space key presses.
3. **Semantic Landmarks & ARIA Markup**:
   - *Source Verified*: Explicit landmark roles (`<header>`, `<main role="main">`, `<aside role="region">`, `<footer>`). Inner modal containers declare `role="dialog"`.
4. **Progress & Live Telemetry**:
   - *Runtime Verified*: Progress bar implements the WAI-ARIA Progressbar pattern (`role="progressbar"`, dynamic `aria-valuenow`, dynamic `aria-valuetext`).
   - *Source Verified*: `#a11yLiveAnnouncer` (`aria-live="polite"`, `aria-atomic="true"`) announces discrete lifecycle status changes and toast messages.
5. **Automated Scanner Results**:
   - *Runtime Verified*: axe-core initial DOM scan against WCAG 2.0/2.1 Level A and AA rules produced **0 critical, serious, or moderate violations**.
6. **Explicit Forensic Verification Boundaries**:
   - *Screen Reader Speech Output*: Synthesized speech output across Windows Narrator, NVDA, and JAWS was not evaluated.
   - *Dynamic State Scans*: Automated axe-core scans covered the initial DOM load; dynamic scans during open drawer or active progress states were not evaluated.
   - *High Contrast Live Rendering*: Forced-colors CSS rules are verified in source code (`base.css:61–108`), but live rendering under active Windows High Contrast OS themes was not evaluated.

---

## 9. Motion, Animation & Transitions Analysis

- **Transition Inventory**:
  - CSS Transitions: `0.15s` hover elevations, `0.15s` button press scaling (`scale(0.97)` to `0.98`), `300ms` CSS variable theme transitions.
  - Keyframe Animations:
    - `modalPop` (`modals.css:98`): `scale(0.96) translateY(8px) -> scale(1) translateY(0)` over `0.25s cubic-bezier(0.16, 1, 0.3, 1)`.
    - `toastSlideIn` (`drawers.css:619`): `translateY(16px) -> translateY(0)` over `0.25s`.
    - `pulse-emerald` (`drawers.css:500`): Box-shadow expansion over `2s` on `#btnDockProcess.ready-pulse`.
    - `pulse` (`tables.css:349`): Opacity shift (`0.6` to `1.0`) over `1.5s` for loading placeholder text.
    - `appleThemeIrisReveal` (`modals.css:494`): Circular clip-path transition over `0.65s`.
  - Drawer Slide: Horizontal translation `translateX(100%) -> translateX(0)` over `0.3s cubic-bezier(0.16, 1, 0.3, 1)`.
- **HIG Motion Evaluation**:
  - *Apple Guidance*: Motion should feel purposeful, physically grounded, and lightweight, communicating spatial relationships and hierarchy without causing delay or disorientation.
  - *Assessment*: Animations are crisp, brief, and purposeful. Modals and drawers slide along natural viewport boundaries, reinforcing spatial origin. Easing curves mimic natural spring physics without bouncing wildly.
  - *Limitation*: As identified in Section 6, the absence of `@media (prefers-reduced-motion)` means users who request minimal motion receive default transitions.

---

## 10. Color, Contrast & State Independence

- **Color Palette & Semantic Tokens**:
  - Primary / Brand: Emerald green (`--accent-emerald: #059669` light, `#10b981` dark).
  - Warnings: Amber (`--accent-amber: #d97706` light, `#f59e0b` dark).
  - Destructive / Errors: Rose red (`--accent-rose: #dc2626` light, `#ef4444` dark).
  - Neutral Base: Slate gray (`--bg-base: #f8fafc` light, `#0b0f17` dark).
- **Non-Color State Indicators**:
  - Incomplete / Complete: Text labels ("Incomplete", "Complete") and SVG icons (clock vs checkmark).
  - Toasts: Distinct icons (checkmark, exclamation triangle, circle X) and bold category titles ("Success", "Warning", "Error").
  - Disabled: Opacity reduction (`0.55`), `cursor: not-allowed`, and `filter: grayscale(0.5)`.
  - Focus: High-visibility focus ring (`--a11y-focus-ring-width: 2px` solid emerald with 2px offset) on all `:focus-visible` elements.
- **State Independence Boundary**:
  - Across all primary workflow paths, color is consistently paired with non-color cues. Complete verification across all edge-case table cells in the Help drawer directory remains unverified.

---

## 11. Desktop Controls & Target Sizing

- **Pointer Target Dimensions**:
  - Primary action button (`#btnDockProcess`): Rendered with `padding: 10px 22px; font-size: 13.5px` (rendered bounds ~40px height, ~180px width), providing a prominent pointer target.
  - Header utility buttons (`#btnToggleTheme`, `#btnOpenSettings`, `#btnOpenHelp`): Rendered at `36×36px` square targets with 8px internal padding.
  - Stepper chips: Rendered as segmented pills with `padding: 6px 14px; min-height: 32px`.
- **Desktop Target Assessment**:
  - Controls are sized appropriately for desktop pointer interaction (mouse/trackpad). Touch target standards (e.g. 44×44pt minimums) are not treated as binding requirements since the application is deployed as a Windows desktop tool.

---

## 12. Legacy HIG Documentation Reconciliation

Forensic review of previous audit claims in legacy documentation:

| Legacy Section / Claim | Status in Current Code | Audit Finding & Correction |
| :--- | :--- | :--- |
| Section 1: "Apple HIG minimalism and Vanilla CSS adoption" (Commit `85`) | **Accurate in intent** | Retained as historical context; reframed from generic "minimalism" to intentional visual hierarchy and whitespace tokens. |
| Section 4: "`app.js` `showToast` method rendering transient overlay messages" | **Inaccurate attribution** | `showToast` is implemented in `js/toast.js` (line 2). `app.js` is solely an application lifecycle bootstrap script. |
| Section 6: "Automated UI test suite fails on accessibility violations... WCAG 2.1 AA Compliance: Present" | **Overclaimed** | Passing initial axe-core scan does not establish complete WCAG 2.1 AA conformance across dynamic states and screen readers. Corrected to reflect empirical test boundaries. |
| General Tone: Repeated claims of "Apple HIG compliant" and "Apple HIG minimalism" | **Subjective judgment** | Replaced with objective evaluation matrix, distinguishing platform-appropriate conventions from cross-platform usability principles. |

---

## 13. Potential Future Remediation (Optional Proposals)

*(These recommendations represent optional architectural proposals for consideration in future development phases; they do not represent defects in the current implementation).*

1. **Operating-System-Aware Reduced-Motion Strategy**:
   - *Proposal*: Introduce an operating-system-aware reduced-motion strategy that suppresses or substitutes nonessential animation for users who request reduced animation through Windows accessibility settings.
   - *Example implementation approach only; not an Apple HIG requirement*:
     ```css
     @media (prefers-reduced-motion: reduce) {
       *, *::before, *::after {
         animation-duration: 0.01ms !important;
         animation-iteration-count: 1 !important;
         transition-duration: 0.01ms !important;
         scroll-behavior: auto !important;
       }
     }
     ```
2. **Inline Form Validation Indicators**:
   - *Proposal*: Consider augmenting pre-flight toast notifications with localized input border highlights and inline validation messages when users attempt to progress with incomplete fields.
3. **Keyboard Shortcut Accelerators**:
   - *Proposal*: Consider binding global accelerator keys (e.g., `Ctrl+,` for Settings, `Ctrl+L` for Diagnostics Logs, `F1` for Help) to expand accessibility for keyboard-only desktop power users.
4. **Dedicated Windows High Contrast Visual Testing**:
   - *Proposal*: Execute visual regression testing under active Windows High Contrast themes (Aquatic/Desert) to verify card border clarity in physical runtime environments.

---

## 14. Verification & Vault Integrity

- **Obsidian Vault Integrity Suite**:
  ```powershell
  pytest tests/test_obsidian_vault_integrity.py -v
  ```
  **Result: 4 passed in 0.08s (100%)**.
  - `test_obsidian_vault_structure_exists` PASSED
  - `test_obsidian_vault_frontmatter_integrity` PASSED
  - `test_obsidian_vault_wikilink_resolution` PASSED
  - `test_obsidian_moc_exists_and_links_all_categories` PASSED
- **Source Code Integrity**:
  - Application repository (`C:\Users\danjo\OneDrive\CVSU GENERATORS`) working tree is completely clean and untouched (`git status`).
- **Whitespace / Formatting Check**:
  - `git diff --check` passes with 0 whitespace errors.

---

## 15. Remaining Uncertainties & Testing Boundaries

1. **Auditory Screen Reader Experience**: While ARIA markup, live regions, and progress bar attributes are verified in source and DOM tests, actual synthesized speech cadence in Windows Narrator/NVDA remains unverified.
2. **Dynamic DOM Accessibility Scans**: The axe-core engine was evaluated against initial page load; dynamic DOM states during active compilation progress and opened slide-over drawers have not been scanned by automated tools.
3. **High Contrast Theme Visuals**: `@media (forced-colors: active)` rules are verified in `base.css`, but visual rendering under active Windows High Contrast themes was not evaluated in this headless Chromium session.
