---
title: "CEIT Templates"
tags:
  - cvsu-generator
  - templates
  - ceit
  - docx
status: active
last_modified: 2026-09-13
source_of_truth:
  - templates/
  - modules/generators/ceit_gen.py
  - modules/generators/generic_doc_gen.py
---

# CEIT Templates

Specifications for the Microsoft Word (`.docx`) templates used by the CEIT document generator.

Related notes:
- [[CvSU Document Generator MOC]]
- [[Template Guidelines]]
- [[CEIT Generator]]

---

## 📋 Registered Native Template Files

| Template Key | Filename | Output Suffix | Target Form |
| :--- | :--- | :--- | :--- |
| `syllabus` | `template_syllabus.docx` | `_SYLLABUS_ACCEPTANCE.docx` | Syllabus Acceptance Sheet |
| `exam_return` | `template_exam_midterm.docx` | `_EXAM_RETURNS_MIDTERM.docx` | Midterm Examination Return Form |
| `exam_return` | `template_exam_finals.docx` | `_EXAM_RETURNS_FINALS.docx` | Finals Examination Return Form |
| `tos` | `template_tos_midterm.docx` | `_TOS_MIDTERM.docx` | Table of Specifications (Midterm) |
| `tos` | `template_tos_finals.docx` | `_TOS_FINALS.docx` | Table of Specifications (Finals) |
| `grade_discussion` | `Midterm-Grade-Discussion_LATEST.docx` | `_GRADE_DISCUSSION_MIDTERM.docx` | Grade Discussion Form (Midterm) |
| `grade_discussion` | `Final-Grade-Discussion_LATEST.docx` | `_GRADE_DISCUSSION_FINALS.docx` | Grade Discussion Form (Finals) |

---

## 🏷️ Native Placeholder Replacement Mapping

The hardcoded `fill_header` implementations in `ceit_gen.py` search for specific labels and inject data immediately after them:
- `INSTRUCTOR:` &rarr; `info.instructor`
- `COURSE / SECTION:` &rarr; `info.course_section`
- `SCHEDULE CODE:` &rarr; `info.schedule_code`
- `SUBJECT:` &rarr; `info.subject`
- `TIME / DAY / ROOM:` &rarr; `info.time_days_room`
- `SEMESTER & A.Y.:` &rarr; `info.semester_ay`

Table row columns (native):
- Column 0: Index / Row number
- Column 1: Student Number
- Column 2: Student Name (`SURNAME, FIRSTNAME M.I.`)
- Subsequent Columns: Signatures or checkbox cells (left blank for physical signing)

---

## ⚙️ Custom Template Recipes (`generic_doc_gen.py`)

Users can import custom Word templates via the UI. These do not rely on hardcoded structural lookups. Instead, they use a declarative JSON "recipe" to map data to the document.

### Header Bindings
Data can be bound in three ways:
1. **Table Cell**: Explicitly targets `table_index`, `row_index`, and `cell_index`.
2. **Paragraph Colon**: Finds a specific `para_index` and injects text after the first colon.
3. **Placeholders**: Direct string replacement. Scans all text runs for tokens like `{{INSTRUCTOR}}`, `{{SUBJECT}}`, `{{COURSE_SECTION}}` and replaces them. The engine performs a two-pass replacement to handle cases where Word fragments placeholder text across multiple XML `<w:r>` runs.

### Roster Table Binding
The recipe defines:
- `table_index`: Which table holds the student list.
- `template_row_index`: Which row to clone for the loop.
- `index_col`, `name_col`, `id_col`: The exact zero-indexed column positions for student data.
