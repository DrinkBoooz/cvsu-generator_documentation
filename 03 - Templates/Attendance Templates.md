---
title: "Attendance Templates"
tags:
  - cvsu-generator
  - templates
  - attendance
  - docx
status: active
last_modified: 2026-09-13
source_of_truth:
  - attendance/
  - modules/generators/attendance_gen.py
---

# Attendance Templates

Specifications for Microsoft Word (`.docx`) attendance templates located in `attendance/`.

Related notes:
- [[CvSU Document Generator MOC]]
- [[Template Guidelines]]
- [[Attendance Generator]]

---

## 📅 Template Layout & Tables

The application utilizes two distinct `.docx` attendance templates stored in the `attendance/` directory, selected dynamically during generation:

### 1. `template lec.docx` (Lecture Only)
Used for standard courses without a laboratory component.
- Features a single row per student.

### 2. `template lab and lec.docx` (Lecture & Laboratory)
Used for courses with integrated laboratory hours.
- Features a slightly different table layout to accommodate both lecture and laboratory attendance tracking if needed.

Both templates consist of two primary sections:

1. **Header Block**:
   - Institutional CvSU header with logo and college title.
   - Metadata rows:
     - Course & Section
     - Schedule Code
     - Subject Description
     - Meeting Day & Time (e.g. `Monday 7:00 AM - 10:00 AM`)
     - Room Assignment
     - Month & Year (e.g. `September 2026`)
     - Instructor

2. **Attendance Grid**:
   - **Column 1**: Row Number (1 to $N$)
   - **Column 2**: Student Name (Alphabetical, Surname first). The generator progressively scales the font size based on character count (`shrink_threshold=32`, `shrink_sz="18"`) to prevent names from overflowing and expanding row height.
   - **Column 3**: Student Number
   - **Columns 4 to 4+N**: Calendar Meeting Dates. The generator dynamically determines the number of meeting sessions based on the schedule, splits the remaining table width evenly across the date columns, and populates headers with the numeric days (e.g., `4`, `11`, `18`).
   - **Summary Columns**: Total Present, Total Absent, Remarks.

---

## ⚙️ Template Selection & Lab Detection Logic

The orchestrator dynamically selects between `template lec.docx` and `template lab and lec.docx` based on the `auto_has_lab` flag.

### Orchestrator Resolution Flow
1. **Explicit Type Override**: If a schedule block `type` explicitly defines `"LAB"` (i.e. `schedule_block['type'] == 'LAB'`), the system forces the use of the lab template.
2. **Subject Title Heuristics**: If no blocks explicitly state `"LAB"`, the orchestrator checks if `"LAB"` or `"LABORATORY"` is present in the `subject_name`. If so, `auto_has_lab = True`.
3. **Known Lab Subjects Verification**: If the subject title does not explicitly contain "LAB", it falls back to checking `KNOWN_LAB_SUBJECT_CODES` (defined in `config_manager.py` / `ceit_directory.py`).
   - `is_known_lab_subject(subject_name)` normalizes the subject code and compares it against a predefined list of computer/technical subjects that require labs (e.g. `DCIT 21`, `COSC 55`, `ITEC 50`).
   - If a match is found, `auto_has_lab = True`.

Based on this resolved `has_lab` flag, the `attendance_gen.py` builder dynamically routes to the correct `.docx` file in the `attendance/` directory.
