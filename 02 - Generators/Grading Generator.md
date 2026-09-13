---
title: "Grading Generator"
tags:
  - cvsu-generator
  - generators
  - grades
  - xlsx
status: active
last_modified: 2026-09-13
source_of_truth:
  - modules/generators/grade_gen.py
  - modules/parsers/ceit_directory.py
  - templates/
---

# Grading Generator

The **Grading Generator** (`modules/generators/grade_gen.py`) populates official CvSU Excel grade sheets while strictly maintaining workbook formula structures, conditional formatting, and multi-tab architectures.

Related notes:
- [[CvSU Document Generator MOC]]
- [[Generators Overview]]
- [[Grading Templates]]
- [[Template Guidelines]]

---

## 📊 Template Selection

The generator automatically selects between two master templates based on the subject's lab hours and department directory:

```mermaid
graph TD
    Class["ClassInfo"] --> LabCheck{"Has Lab Hours or Known Lab Code?"}
    LabCheck -->|Yes| LabTpl["GRADING_LECTURE_LAB_TEMPLATE.xlsx<br>(4 Tabs: Lecture, Lab, Consolidated, Grading Sheet)"]
    LabCheck -->|No| LecTpl["GRADING_LECTURE_TEMPLATE.xlsx<br>(2 Tabs: Lecture, Grading Sheet)"]
```

Faculty members can also override the auto-detected template type via the dropdown in Step 3 of the UI.

---

## 🔒 Formula & Cell Integrity Rules

1. **`openpyxl` with `data_only=False`**:
   - The template workbook is loaded with formulas enabled so Excel formula definitions (`SUM`, `AVERAGE`, `VLOOKUP`, transmutation formulas) are preserved when the file is saved.
2. **Student Row Insertion**:
   - Locates student entry coordinates (starting at Row 11).
   - Writes Student Number into Column B and Student Name into Column C.
   - Preserves formula references in grading columns across all worksheet tabs.
3. **Metadata Headers**:
   - Updates official college banner, course code, section, semester, academic year, and instructor name.
4. **Signatures**:
   - Injects the instructor's name into the instructor signature block and department chairperson into the reviewer block.

---

## 📁 Output Location & Naming

Grading spreadsheets are saved in the root folder of each class section:
```text
<Output Directory>/<Course_Section>/<Course_Sec>_<SchedCode>_GRADING_SHEET.xlsx
```
Example:
```text
BSCS 1-4/BSCS1-4_202612040_GRADING_SHEET.xlsx
```
