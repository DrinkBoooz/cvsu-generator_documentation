---
title: "Generator Pipeline"
tags:
  - cvsu-generator
  - architecture
  - pipeline
  - parsers
status: active
last_modified: 2026-09-13
source_of_truth:
  - modules/parsers/schedule_parser.py
  - modules/parsers/roster_parser.py
  - modules/services/validator.py
  - modules/models/schedule.py
---

# Generator Pipeline

The **Generator Pipeline** is responsible for transforming raw input spreadsheets into clean domain models, matching class sections with their corresponding student rosters, and dispatching tasks to document generators.

Related notes:
- [[CvSU Document Generator MOC]]
- [[System Architecture]]
- [[Orchestrator Lifecycle]]
- [[Generators Overview]]
- [[Input File Conventions]]

---

## 🔄 End-to-End Pipeline Stages

```mermaid
graph TD
    SchedXLS["Schedule (.xls/.xlsx)"] -->|modules/parsers/schedule_parser.py| ClassInfoList["ClassInfo Domain Objects"]
    RosterFiles["Student Rosters (.xlsx/.csv)"] -->|modules/parsers/roster_parser.py| StudentRecords["Student Records List"]
    
    ClassInfoList --> Matcher["Class & Roster Validator<br>(modules/services/validator.py)"]
    StudentRecords --> Matcher

    Matcher --> LabDetect["Lab Subject Classifier<br>(config_manager / overrides)"]

    LabDetect -->|Lecture Only| Dis1["GradeGen (Lecture Template)"]
    LabDetect -->|Lecture & Lab| Dis2["GradeGen (Lecture+Lab Template)"]
    LabDetect --> Dis3["AttendanceGen (Calendar Dates)"]
    LabDetect --> Dis4["CEITGen (Registered Forms)"]
```

---

## 1. Schedule Parsing (`modules/parsers/schedule_parser.py`)
- Reads the official Faculty Timetable Schedule using `openpyxl` or `xlrd`.
- Identifies:
  - Instructor full name and college title.
  - Semester and Academic Year (e.g. `1st Semester A.Y. 2026-2027`).
  - Table grid rows: Course Code, Course Title, Section (`BSCS 1-4`), Schedule Code (`202612040`), Room, Contact Hours (Lec / Lab), Days of week (`M`, `T`, `W`, `TH`, `F`, `S`), and Meeting Times.
- Cleans and consolidates multi-block schedules (e.g. separate Lecture and Lab sessions for the same class section) into single `ClassInfo` instances.

## 2. Roster Parsing (`modules/parsers/roster_parser.py`)
- Reads student rosters exported from the CvSU Registrar portal.
- Extracts:
  - Student Full Name (using heuristic or explicit column configuration).
  - Student Number (using heuristic or explicit column configuration).
- Strips portal metadata columns, empty trailing rows, and normalizes capitalization.
- Pairs with schedule blocks using either:
  1. Standard naming pattern (e.g. `{Course/Sec} List of Students for {ScheduleCode}-{Subject}.xlsx`).
  2. Manual `linked_schedule_code` configured via the UI.

## 3. Validation & Matching (`modules/services/validator.py`)
- Takes the parsed `ClassInfo` models and the parsed rosters and validates the intersection.
- Determines which rosters are orphaned (no matching schedule) and which schedule blocks lack a roster.
- Ensures all minimum requirements for generation (valid instructor, valid student rows, valid schedule matrix) are met before passing data to the Orchestrator.

## 4. Lab Classification
- Analyzes syllabus lab hours and cross-references against `KNOWN_LAB_SUBJECT_CODES` (defined in the central `cvsu_parser_config.json`).
- If `"LAB"` or `"LABORATORY"` appears in the subject string, it is automatically flagged.
- Flags classes requiring the 4-tab Excel grading workbook (`GRADING_LECTURE_LAB_TEMPLATE.xlsx`) versus standard lecture courses (`GRADING_LECTURE_TEMPLATE.xlsx`).
- Provides fallback user overrides configured in Step 3 of the UI (stored as `type_overrides` passed to the Orchestrator).
