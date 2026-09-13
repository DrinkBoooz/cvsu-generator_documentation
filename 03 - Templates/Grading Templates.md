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

## 🔒 Protected Coordinate Conventions

- **Row 1 &ndash; 10**: College crest, campus name, department header, and instructor information.
- **Row 11**: Starting row for student data entry.
  - Column B: Student Number (`2026XXXXX`)
  - Column C: Student Full Name (`SURNAME, FIRSTNAME M.I.`)
- **Computed Formula Ranges**: Must never be overwritten by the generator; formula links across worksheets are automatically evaluated by Excel upon opening.
