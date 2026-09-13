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

## State Dictionary

| State | Owner | Source | Consumers | Mutation | Lifecycle |
| ----- | ----- | ------ | --------- | -------- | --------- |
| `schedule_path` | `orchestrator.py` | UI File Drop / Select | Config Manager, Parsers | Re-assigned on new ingestion | Ephemeral per batch |
| `output_dir` | `orchestrator.py` | Computed from `ClassInfo` | Generators | Created per valid class | Ephemeral per generation |
| `rosters` | `state.js` (`AppState`) | File Input | UI rendering, PyWebView | Cleared/Updated on drop | Ephemeral UI Session |
| `roster_configs` | `config_manager.py` | `parser_settings.json` | Orchestrator, Parsers | UI settings save / Reset | Persistent across sessions |
| `_is_processing` | `orchestrator.py` | Thread state | Stepper UI, PyWebView | Toggled at start/end/cancel | Thread Execution |
| `_is_window_closed` | `orchestrator.py` | PyWebView Event | Generation loop | Set on window close | Application Lifecycle |
| `_cancel_event` | `orchestrator.py` | `threading.Event` | Generation loop, UI | Set by Cancel button | Per generation job |
| `_lock` | `orchestrator.py` | `threading.Lock` | File writers | Acquired during save | Atomic write scope |
