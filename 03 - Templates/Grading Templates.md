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
The generator enforces specific capacity limits based on the template type to prevent overwriting static summary formulas:
- **Lecture Template (`GRADING_LECTURE_TEMPLATE.xlsx`)**:
  - Starts data entry at **Row 11**.
  - Maximum capacity: **60 students**.
- **Lecture + Lab Template (`GRADING_LECTURE_LAB_TEMPLATE.xlsx`)**:
  - Starts data entry at **Row 12**.
  - Maximum capacity: **40 students**.

If a class roster exceeds the maximum capacity, the orchestrator triggers clamping logic (e.g. `cleaned_students[:max_capacity]`), logging a warning and safely ignoring overflow students rather than corrupting the template.

### Signature Anchor Heuristics
For laboratory templates, the instructor signature cell location can shift depending on formatting. The generator uses a dynamic search heuristic:
1. It anchors the search at cell `BI57`.
2. It sweeps a bounding box from row `52` to `62` (offset -5 to +5) and columns `41` to `66` (offset -20 to +5).
3. If it finds a cell containing the word "instructor" (case-insensitive), it places the instructor's name exactly 3 rows above the matched cell.
4. If not found, it falls back gracefully with a logged warning.
