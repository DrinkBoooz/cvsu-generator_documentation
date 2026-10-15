---
title: "Application Data Flow"
tags:
  - cvsu-generator
  - architecture
  - data-flow
  - smart-ingest
  - roster-diff
status: active
last_modified: 2026-10-16
source_of_truth:
  - modules/services/orchestrator.py
  - modules/parsers/roster_parser.py
  - modules/parsers/smart_ingest.py
  - modules/models/workspace.py
  - modules/common/config_manager.py
---

# Application Data Flow

This document maps how data flows through the application from user input to generated documents, including Smart Ingest routing, mid-semester roster diffing, and semester workspace lifecycle state.

Related notes:
- [[CvSU Document Generator MOC]]
- [[System Architecture]]
- [[Orchestrator Lifecycle]]
- [[State Management]]
- [[Persistent State Architecture]]

---

## 🔀 Data Flow Pipeline

Data follows a strictly unidirectional flow from the frontend through the backend parsers, resolvers, and generators:

### 1. Ingestion & Smart Ingest Router (`smart_ingest.py` & Bridge)
- **User Actions**: The user drags and drops single files, mixed file batches, or folders into the UI, or selects them via native OS dialogs.
- **Smart Ingest Classification**: `SmartIngestClassifier` inspects file extensions and content structures deterministically:
  - `.cvsu-workspace.json`: Loads or imports the complete semester workspace state.
  - `.cvsu-org.json`: Imports and applies organizational hierarchy and signatory slots.
  - `.ics`: Ingests institutional academic calendar and typhoon/holiday exceptions into `CalendarResolver`.
  - `.docx`: Inspects or stages custom CEIT document templates.
  - `.xlsx` / `.xls` / `.csv`: Differentiates faculty schedules vs student rosters vs grading templates based on internal cell structures and filename heuristics.
- **PyWebView Bridge**: `smart_ingest_files(file_paths)` applies detected items directly to the application state, triggering automated re-validation.

### 2. Parsing & Mid-Semester Diffing (`roster_parser.py`)
- **Extraction**: `load_students()` normalizes student records from Excel or CSV into standardized `[(Name, StudentID), ...]` tuples, resolving corrupted encoding via `_fix_encoding()`.
- **Roster Diffing Engine**: Mid-semester class list updates are processed via `compute_roster_diff(old_students, new_students)`:
  - Deterministically categorizes records into `added` (newly enrolled), `removed` (dropped / withdrawn), `modified` (spelling or ID corrections), and `unchanged`.
  - The UI presents a visual diff modal with green (+ Added) and red (- Dropped) badges before 1-click update.

### 3. Orchestration & Configuration (`orchestrator.py` & `config_manager.py`)
- **Metadata Resolution**: The orchestrator extracts schedule codes, subject codes, and course/sections.
- **`ClassInfo` Construction**: An internal `ClassInfo` structure is populated with `course`, `schedule`, `semester`, `room`, `instructor`, and `subject`.
- **Configuration & Workspace Context**: `config_manager.py` enriches generation with institutional settings (`DEFAULT_CEIT_PREFIX_MAP`, `schedule_config`), active `OrganizationPreset`, active `CalendarResolver`, and the active `TemplateSet`.

### 4. Dispatch to Generators
- Based on the requested engines (`ceit`, `attendance`, `grades`), the orchestrator passes cleaned student records, meeting dates, and metadata to:
  - `ceit_gen.py` (Word `.docx` academic forms)
  - `attendance_gen.py` (Word `.docx` attendance tracking sheets with dynamic calendar resolution)
  - `grade_gen.py` (Excel `.xlsx` electronic grading sheets)
  - Optional headless PDF compilation via `pdf_exporter.py`

### 5. Telemetry & Output
- As documents are saved to disk, the orchestrator updates the UI via `window.onGenerationProgress`.
- Output folders (`CEIT_Forms`, `Attendance`) are dynamically created inside a root `<Course_Sec>` directory based on the resolved `ClassInfo`.
