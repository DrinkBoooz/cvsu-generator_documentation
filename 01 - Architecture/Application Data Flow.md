---
title: "Application Data Flow"
tags:
  - cvsu-generator
  - architecture
  - data-flow
status: active
last_modified: 2026-09-13
source_of_truth:
  - modules/services/orchestrator.py
  - modules/parsers/roster_parser.py
  - modules/common/config_manager.py
---

# Application Data Flow

This document maps how data flows through the application from user input to generated documents.

Related notes:
- [[CvSU Document Generator MOC]]
- [[System Architecture]]
- [[Orchestrator Lifecycle]]
- [[State Management]]

---

## 🔀 Data Flow Pipeline

Data follows a strictly unidirectional flow from the frontend through the backend parsers and generators:

### 1. Ingestion (Frontend -> Backend Bridge)
- **User Actions**: The user drags and drops a folder containing class rosters (`.xlsx` or `.csv`) into the UI or selects via a file dialog. 
- **PyWebView Bridge**: The `ScriptAPI` mixins intercept the dropped file paths and forward them to the backend orchestrator.

### 2. Parsing & Normalization (`roster_parser.py`)
- The `load_students` function opens the files.
- **Encoding & Column Detection**: It resolves potential character encoding issues (`_fix_encoding`) and dynamically detects student ID and name columns using `_is_id_header` and `_is_name_header`.
- **Extraction**: The students are normalized into a standardized `[(Name, StudentID), ...]` tuple structure regardless of whether they were from CSV or Excel.

### 3. Orchestration & Configuration (`orchestrator.py` & `config_manager.py`)
- **Metadata Resolution**: The orchestrator extracts the schedule code, subject code, and course/section from the filenames.
- **`ClassInfo` Construction**: An internal `ClassInfo` or dictionary object is populated containing `course`, `schedule`, `semester`, `room`, `instructor`, and `subject`.
- **Configuration Lookup**: `config_manager.py` provides the current institutional settings (e.g. `DEFAULT_CEIT_PREFIX_MAP`, `DEFAULT_SCHEDULE_CONFIG`) to enrich the metadata.

### 4. Dispatch to Generators
- Based on the requested engines (`ceit`, `attendance`, `grades`) and dynamic lab detection logic (e.g. `KNOWN_LAB_SUBJECT_CODES`), the orchestrator passes the cleaned student tuples and `ClassInfo` to the specific generators:
  - `ceit_gen.py` (Word `.docx`)
  - `attendance_gen.py` (Word `.docx`)
  - `grade_gen.py` (Excel `.xlsx`)

### 5. Telemetry & Output
- As documents are saved to disk, the orchestrator updates the UI via `window.onGenerationProgress`.
- Output folders (`CEIT_Forms`, `Attendance`) are dynamically created inside a root `<Course_Sec>` directory based on the resolved `ClassInfo`.
