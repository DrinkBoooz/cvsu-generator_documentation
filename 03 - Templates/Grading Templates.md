---
title: "Grading Templates"
tags:
  - cvsu-generator
  - templates
  - grades
  - xlsx
status: active
last_modified: 2026-09-13
source_of_truth:
  - templates/
  - modules/generators/grade_gen.py
---

# Grading Templates

Specifications for Microsoft Excel (`.xlsx`) official grading sheet workbooks.

Related notes:
- [[CvSU Document Generator MOC]]
- [[Template Guidelines]]
- [[Grading Generator]]

---

## 📊 Master Workbook Files

### 1. `GRADING_LECTURE_TEMPLATE.xlsx`
Used for courses with pure lecture contact hours:
- **Sheet 1 (`Lecture`)**: Class attendance, quizzes, assignments, midterm examination, and final examination breakdown.
- **Sheet 2 (`Grading Sheet`)**: Official institutional summary tab containing transmuted numeric grades (1.00 to 5.00), passing/failing status, and signature lines.

### 2. `GRADING_LECTURE_LAB_TEMPLATE.xlsx`
Used for technical and computer courses featuring integrated laboratory components:
- **Sheet 1 (`Lecture`)**: 60% standard lecture weight.
- **Sheet 2 (`Laboratory`)**: 40% laboratory activities and practical exams.
- **Sheet 3 (`Consolidated`)**: Formula-linked consolidation between Lecture and Lab percentages.
- **Sheet 4 (`Grading Sheet`)**: Official institutional summary tab with computed final marks.

---

## 🔒 Protected Coordinate Conventions & Generator Boundaries

The `grade_gen.py` module strictly enforces bounds to protect Excel formula evaluations and formatting.

- **Row 1 &ndash; 10**: College crest, campus name, department header, and instructor information.
- **Computed Formula Ranges**: Must never be overwritten by the generator; formula links across worksheets are automatically evaluated by Excel upon opening.

### Capacity Clamping & Row Boundaries
The generator dynamically discovers and enforces capacity limits based on verified template formula ranges:
- **Lecture Template (`GRADING_LECTURE_TEMPLATE.xlsx`)**:
  - Starts data entry at **Row 11** (`recipe.roster_binding.first_data_row_index`).
  - Pre-formatted formula rows: Rows 11&ndash;50.
  - Maximum capacity: **40 students** (`capacity_limit = 40`).
- **Lecture + Lab Template (`GRADING_LECTURE_LAB_TEMPLATE.xlsx`)**:
  - Starts data entry at **Row 12** (`recipe.roster_binding.first_data_row_index`).
  - Pre-formatted formula rows: Rows 12&ndash;51.
  - Maximum capacity: **40 students** (`capacity_limit = 40`).

If a class roster exceeds the maximum capacity, `grade_gen.py` safely clamps the roster to `capacity_limit` (e.g. `students[:capacity_limit]`), logging a warning and never corrupting the template footer formulas.

### Structural Signature Geometry Discovery (Mutation M8)
Rather than using fragile hardcoded coordinates or naive relative row offsets (such as `label row - 3`), `XlsxTemplateInspector` discovers the signature target from structural merged cell geometry:
1. It locates the merged label block containing `"INSTRUCTOR"` (e.g. `BI60:BR62` in Lecture, `AO62:AX64` in Lab, `J59:S61` in Consolidated).
2. It scans all merged cell ranges in the worksheet for the structural signature block directly above it (`min_col == label.min_col`, `max_col == label.max_col`, and `max_row == label.min_row - 1`).
3. It dynamically resolves the top-left coordinate of that merged box (`BI57`, `AO59`, `J56`), rendering discovery completely resilient to row additions or structural repositioning (Mutation M8).
