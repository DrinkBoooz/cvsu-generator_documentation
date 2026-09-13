---
title: "State Management"
tags:
  - cvsu-generator
  - architecture
  - state
status: active
last_modified: 2026-09-13
source_of_truth:
  - modules/common/config_manager.py
  - executable_test/js/state.js
  - executable_test/js/stepper.js
---

# State Management

This document outlines how the application manages persistent configuration state and ephemeral UI session state.

Related notes:
- [[CvSU Document Generator MOC]]
- [[System Architecture]]
- [[UI Architecture]]

---

## 💾 Persistent Configuration State

The application configuration, which includes departmental aliases, subject prefixes, and known lab subjects, is managed centrally by `ParserConfigManager` in `modules/common/config_manager.py`.

### 1. Merging Strategy
- The application ships with factory defaults (`DEFAULT_CEIT_PREFIX_MAP`, `DEFAULT_ROSTER_KEYWORDS`, etc.).
- When the application launches, `ParserConfigManager` attempts to load a `parser_settings.json` file from the user's `APPDATA/CVSU_Generators/config` directory.
- It performs a deep merge, prioritizing user overrides while falling back to factory defaults for any missing keys.

### 2. State Observers
- `ParserConfigManager` implements a simple observer pattern (`register_listener` and `_notify_listeners`).
- When the configuration is saved, reset, or imported, all registered callbacks are triggered, allowing the UI or running services to reactively update.

---

## 🖥️ Ephemeral UI Session State

The frontend JavaScript separates the UI state logic from the DOM manipulation.

### 1. Global Application State (`state.js`)
- The `AppState` class acts as the single source of truth for the current session.
- It holds references to:
  - The list of loaded student rosters (`files`).
  - Currently discovered course/section metadata (`classes`).
  - Active toggle states for the engines (e.g. `generateCEIT`, `generateAttendance`).
- The `updateUI` method is responsible for re-rendering file lists, detected classes, and warning badges based purely on the current data within `AppState`.

### 2. Workflow Progression (`stepper.js`)
- The visual progress of the user is managed by the `StepperState` class.
- The UI features 6 distinct steps (Ingestion, Configuration, Review, Generation, etc.).
- State transitions (e.g. advancing from Step 2 to Step 3) trigger DOM updates to highlight active "chips" and animated connectors.
- The stepper strictly blocks progression unless specific state validation criteria are met (e.g., at least one valid roster must be loaded in `AppState` before moving to generation).
