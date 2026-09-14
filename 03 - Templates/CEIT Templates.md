---
title: "CEIT Templates"
tags:
  - cvsu-generator
  - templates
  - ceit
  - docx
status: active
last_modified: 2026-09-15
source_of_truth:
  - templates/
  - modules/generators/ceit_gen.py
  - modules/generators/generic_doc_gen.py
  - modules/parsers/template_inspector.py
  - modules/parsers/recipe_validator.py
  - modules/models/recipe.py
---

# CEIT Templates

Specifications for the Microsoft Word (`.docx`) templates used by the CEIT document generator and custom generic pipeline.

Related notes:
- [[CvSU Document Generator MOC]]
- [[Template Guidelines]]
- [[CEIT Generator]]
- [[Generic Document Generator]]
- [[Template Discovery Pipeline]]

---

## 📋 Registered Native Template Files

| Template Key | Filename | Output Suffix | Target Form | Profile ID |
| :--- | :--- | :--- | :--- | :--- |
| `syllabus` | `template_syllabus.docx` | `_SYLLABUS_ACCEPTANCE.docx` | Syllabus Acceptance Sheet (VPAA-QF-12) | `syllabus` (`academic_docx`) |
| `exam_midterm` | `template_exam_midterm.docx` | `_EXAM_RETURNS_MIDTERM.docx` | Midterm Examination Return Form (CEIT-QF-03) | `exam_returns` (`academic_docx`) |
| `exam_finals` | `template_exam_finals.docx` | `_EXAM_RETURNS_FINALS.docx` | Finals Examination Return Form (CEIT-QF-03) | `exam_returns` (`academic_docx`) |
| `tos_midterm` | `template_tos_midterm.docx` | `_TOS_MIDTERM.docx` | Table of Specifications (Midterm) | `tos` (`academic_docx`) |
| `tos_finals` | `template_tos_finals.docx` | `_TOS_FINALS.docx` | Table of Specifications (Finals) | `tos` (`academic_docx`) |
| `grade_midterm` | `Midterm-Grade-Discussion_LATEST.docx` | `_GRADE_DISCUSSION_MIDTERM.docx` | Grade Discussion Form (Midterm) | `grade_discussion` (`academic_docx`) |
| `grade_finals` | `Final-Grade-Discussion_LATEST.docx` | `_GRADE_DISCUSSION_FINALS.docx` | Grade Discussion Form (Finals) | `grade_discussion` (`academic_docx`) |

---

## 🏷️ Recipe-Driven Binding Model

Native CEIT generators contain no heuristic template discovery or positional fallback; binding targets are supplied by the validated recipe. `DocxTemplateInspector` scans the physical template table cells and paragraphs for semantic labels (using `SemanticRegistry`), emitting candidate observations that `RecipeValidator` verifies and compiles into immutable `ValidatedTemplateRecipe` instances.

### Semantic Labels Observed by Inspector
- `INSTRUCTOR:` / `INSTRUCTOR’S NAME/SIGNATURE` &rarr; mapped to `instructor` (`shrink_threshold=30`)
- `COURSE / SECTION:` / `COURSE / YEAR / SECTION` &rarr; mapped to `course_section`
- `SCHEDULE CODE:` &rarr; mapped to `schedule_code`
- `SUBJECT:` / `SUBJECT CODE / TITLE` &rarr; mapped to `subject` (`shrink_threshold=40`)
- `TIME / DAY / ROOM:` &rarr; mapped to `time_days_room`
- `SEMESTER & A.Y.:` / `SEMESTER / ACADEMIC YEAR` &rarr; mapped to `semester_ay`

### Roster Table Column Bindings (Native Academic Forms)
The roster table is identified dynamically by `DocxTemplateInspector` by scanning header text (`name of student`, `student number`, `signature`):
- **Syllabus (`template_syllabus.docx`)**:
  - `table_index`: 1, `first_data_row_index`: 1
  - `index_col`: 0 (Item No.), `id_col`: 1 (Student No.), `name_col`: 2 (Name of Student)
- **Exam Returns (`template_exam_midterm.docx`, `template_exam_finals.docx`)**:
  - `table_index`: 1, `first_data_row_index`: 1
  - `name_col`: 0 (Name of Student), `id_col`: 1 (Student Number)
- **Table of Specifications (`template_tos_midterm.docx`, `template_tos_finals.docx`)**:
  - `table_index`: 1, `first_data_row_index`: 1
  - `name_col`: 0 (Name of Student), `id_col`: 1 (Student Number)
- **Grade Discussion (`Midterm-Grade-Discussion_LATEST.docx`, `Final-Grade-Discussion_LATEST.docx`)**:
  - `table_index`: 1, `first_data_row_index`: 1
  - `name_col`: 0 (Name of Student), `id_col`: 1 (Student ID Number)

Subsequent columns (signatures, dates, grades, remarks) are preserved intact from the template and left blank for physical entry.

---

## ⚙️ Custom Template Recipes (`generic_doc_gen.py`)

Users can import custom Word templates via the application interface. Custom templates do not require predefined layouts. Instead, `ConfigurableDocumentGenerator` validates serialized recipe dictionaries through `RecipeValidator.validate_dict(..., "custom_docx")` when a validated recipe object is not already supplied.

`validate_dict()` strictly enforces `schema_version == 2` and rejects externally supplied `_construction_token`.

### Supported Binding Types
1. **Table Cell (`table_cell`)**: Targets an explicit cell coordinate via `(table_index, row_index, cell_index)`.
2. **Paragraph Colon (`paragraph_colon`)**: Locates a specific `para_index` and injects text immediately following the first colon separator.
3. **Placeholders (`placeholders`)**: Scans all text runs for tokens such as `{{INSTRUCTOR}}`, `{{SUBJECT}}`, `{{COURSE_SECTION}}`. Uses a two-pass replacement engine to resolve placeholder text fragmented across multiple XML `<w:r>` runs.

### Custom Roster Table Bindings
The custom recipe defines:
- `table_index`: Zero-based index of the table hosting the student list.
- `first_data_row_index`: Zero-based row index of the prototype row to clone.
- `index_col`: Optional column index for sequential student numbers (`1..N`).
- `name_col`: Zero-based column index for student full names.
- `id_col`: Zero-based column index for student ID numbers.
