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
---

# CEIT Templates

Specifications for the Microsoft Word (`.docx`) templates used by the CEIT document generator.

Related notes:
- [[CvSU Document Generator MOC]]
- [[Template Guidelines]]
- [[CEIT Generator]]

---

## 📋 Registered Template Files

| Template Key | Filename | Output Suffix | Target Form |
| :--- | :--- | :--- | :--- |
| `syllabus` | `template_syllabus.docx` | `_SYLLABUS_ACCEPTANCE.docx` | Syllabus Acceptance Sheet |
| `exam_return` | `template_exam_return.docx` | `_EXAM_RETURNS_MIDTERM.docx` | Midterm Examination Return Form |
| `exam_return` | `template_exam_return.docx` | `_EXAM_RETURNS_FINALS.docx` | Finals Examination Return Form |
| `tos` | `template_tos.docx` | `_TOS_MIDTERM.docx` | Table of Specifications (Midterm) |
| `tos` | `template_tos.docx` | `_TOS_FINALS.docx` | Table of Specifications (Finals) |
| `grade_discussion` | `template_grade_discussion.docx` | `_GRADE_DISCUSSION_MIDTERM.docx` | Grade Discussion Form (Midterm) |
| `grade_discussion` | `template_grade_discussion.docx` | `_GRADE_DISCUSSION_FINALS.docx` | Grade Discussion Form (Finals) |

---

## 🏷️ Placeholder Replacement Mapping

The `fill_header` implementation replaces standard metadata labels:
- `INSTRUCTOR:` &rarr; `info.instructor`
- `COURSE / SECTION:` &rarr; `info.course_section`
- `SCHEDULE CODE:` &rarr; `info.schedule_code`
- `SUBJECT:` &rarr; `info.subject`
- `TIME / DAY / ROOM:` &rarr; `info.time_days_room`
- `SEMESTER & A.Y.:` &rarr; `info.semester_ay`

Table row columns:
- Column 0: Index / Row number
- Column 1: Student Number
- Column 2: Student Name (`SURNAME, FIRSTNAME M.I.`)
- Subsequent Columns: Signatures or checkbox cells (left blank for physical signing)
