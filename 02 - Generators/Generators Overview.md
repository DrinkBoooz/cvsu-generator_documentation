---
title: "Generators Overview"
tags:
  - cvsu-generator
  - generators
  - overview
status: active
last_modified: 2026-09-13
source_of_truth:
  - modules/generators/
  - modules/services/orchestrator.py
---

# Generators Overview

The document generation engine is decoupled into **three generator families**, each responsible for a distinct document format, template mechanism, and academic output standard.

Related notes:
- [[CvSU Document Generator MOC]]
- [[System Architecture]]
- [[CEIT Generator]]
- [[Attendance Generator]]
- [[Grading Generator]]
- [[Template Guidelines]]

---

## 📊 Summary of Generator Families

| Family | Template Folder | Generator Module | Target Output Subfolder | File Naming Pattern |
| :--- | :--- | :--- | :--- | :--- |
| **[[CEIT Generator]]** (.docx) | `templates/` | `modules/generators/ceit_gen.py` | `<Output>/<Course_Sec>/CEIT_Forms/` | `<Course_Sec>_<SchedCode>_<SUFFIX>.docx` |
| **[[Attendance Generator]]** (.docx) | `attendance/` | `modules/generators/attendance_gen.py` | `<Output>/<Course_Sec>/Attendance/` | `<Course_Sec>_<SchedCode>_ATTENDANCE_<Day>_<Month>.docx` |
| **[[Grading Generator]]** (.xlsx) | `templates/` | `modules/generators/grade_gen.py` | `<Output>/<Course_Sec>/` | `<Course_Sec>_<SchedCode>_GRADING_SHEET.xlsx` |

---

## ⚙️ Execution Strategy in Orchestrator

When `process_all` runs for a given class section:
1. **CEIT Forms**: Iterates through all generator instances provided by `GeneratorFactory.get_all()`, producing 7 official Word documents.
2. **Attendance Sheets**: Analyzes class meeting days and generates 1 Word document per month covering all semester calendar dates.
3. **Grading Sheet**: Selects the appropriate workbook (`GRADING_LECTURE_TEMPLATE.xlsx` or `GRADING_LECTURE_LAB_TEMPLATE.xlsx`), populates enrolled students, instructor signatures, and formula tabs, and saves the final workbook in the class root directory.
