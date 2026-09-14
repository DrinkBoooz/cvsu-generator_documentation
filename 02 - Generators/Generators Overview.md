---
title: "Generators Overview"
tags:
  - cvsu-generator
  - generators
  - overview
  - comparison-matrix
status: active
last_modified: 2026-09-15
source_of_truth:
  - modules/generators/
  - modules/services/orchestrator.py
  - modules/services/template_recipe_service.py
  - modules/parsers/recipe_validator.py
  - modules/parsers/template_inspector.py
  - modules/models/recipe.py
---

# Generators Overview

The document generation engine is partitioned into distinct generator engines, responsible for specific academic document formats, template mechanisms, and institutional compliance standards.

Related notes:
- [[CvSU Document Generator MOC]]
- [[System Architecture]]
- [[Generator Pipeline]]
- [[Template Discovery Pipeline]]
- [[CEIT Generator]]
- [[Attendance Generator]]
- [[Grading Generator]]
- [[Generic Document Generator]]
- [[Template Guidelines]]

---

## 🏛️ Authoritative Recipe-Driven Architecture

The application uses immutable `ValidatedTemplateRecipe` objects constructed exclusively by `RecipeValidator` from inspector-produced `RawTemplateRecipeCandidate` objects and resolved/cached by `TemplateRecipeResolver`. 

No generator-owned hardcoded template cell coordinates are used for recipe-bound field placement; structural targets are supplied by the validated recipe. Native CEIT generators contain no heuristic template discovery or positional fallback; binding targets are supplied by the validated recipe.

```text
Template file
    ↓
DocxTemplateInspector / XlsxTemplateInspector
    ↓
RawTemplateRecipeCandidate
    ↓
RecipeValidator.validate(...)
    ↓
ValidatedTemplateRecipe
    ↓
TemplateRecipeResolver cache / return
    ↓
Generator
    ↓
Generated output
```

### Key Invariants
1. **Inspector Non-Authority**: `DocxTemplateInspector` and `XlsxTemplateInspector` inspect physical template layout, measurements, and labels to emit unvalidated `RawTemplateRecipeCandidate` objects. Inspectors never construct or return `ValidatedTemplateRecipe`.
2. **Validator Sole Authority**: `RecipeValidator.validate()` is the sole entry point permitted to construct `ValidatedTemplateRecipe` (guarded by `_PRIVATE_CONSTRUCTION_SENTINEL` and Schema Version 2).
3. **Resolution & Caching**: `TemplateRecipeResolver` maintains a 3-tuple cached index `(abs_path, profile_id, sha256_fingerprint)`. When a template changes on disk, the SHA-256 fingerprint mismatch triggers automatic invalidation and fresh inspection/validation.
4. **Pure Recipe Execution**: `DocumentGenerator` and `GradeGenerator` consume `ValidatedTemplateRecipe` instances exclusively and execute without positional or coordinate heuristics.

---

## 🏷️ Verification Taxonomy

To maintain source-of-truth fidelity, all specifications and behaviors in this documentation are classified under the following verification tiers:

- **Source-verified**: Directly confirmed by inspecting active application code in the `dev` branch.
- **Test-verified**: Directly validated by passing automated unit, regression, or parity test suites in `tests/` that directly exercise the component or generator.
- **Factory-registration test-verified**: Directly validated by automated tests asserting generator factory registration, instantiation, and template existence without executing `.generate()`.
- **Template-verified**: Directly inspected and extracted from the physical production `.docx` or `.xlsx` template files.
- **Runtime-verified**: Separately verified through an actual application/runtime execution (not inferred merely from unit test execution).
- **Documentation-derived**: Derived from project specifications, schemas, or architectural design notes where direct executable assertions are secondary.
- **Not verified**: Open items, untestable external dependencies, or unconfirmed behaviors.

---

## 📊 Summary of Generator Families

| Family | Template Folder | Generator Module | Target Output Subfolder | File Naming Pattern |
| :--- | :--- | :--- | :--- | :--- |
| **[[CEIT Generator]]** (.docx) | `templates/` | `modules/generators/ceit_gen.py` | `<Output>/<Course_Sec>/CEIT_Forms/` | `<Course_Sec>_<SchedCode>_<SUFFIX>.docx` |
| **[[Attendance Generator]]** (.docx) | `attendance/` | `modules/generators/attendance_gen.py` | `<Output>/<Course_Sec>/Attendance/` | `<Course_Sec>_<SchedCode>_ATTENDANCE_<Day>_<Month>.docx` |
| **[[Grading Generator]]** (.xlsx) | `templates/` | `modules/generators/grade_gen.py` | `<Output>/<Course_Sec>/Grades/` | `<Course_Sec>_<SchedCode>_GRADING_SHEET.xlsx` |
| **[[Generic Document Generator]]** (.docx) | Custom paths | `modules/generators/generic_doc_gen.py` | `<Output>/<Course_Sec>/CEIT_Forms/` | `<Course_Sec>_<SchedCode>_<SUFFIX>.docx` |

---

## 📋 Master Generator Comparison Matrix

The following comparison matrix covers all 12 document generator targets supported by the application. Each target was audited against the project's 23-attribute checklist, and any unavailable evidence (such as lack of direct automated generation tests for native templates) is explicitly marked:

| # | Generator Target | Entry Point | Format | Template / Recipe Profile | Inputs | Target Output | Error Handling | Automated Tests | Verification Status |
| :-: | :--- | :--- | :-: | :--- | :--- | :--- | :--- | :--- | :--- | :-: |
| **1** | **SyllabusGenerator** | `SyllabusGenerator.generate()` | `.docx` | `template_syllabus.docx`<br>Profile: `syllabus` (`academic_docx`) | `ClassInfo` (instructor, course, sched, subject, time, semester, students) | `<Output>/<Course_Sec>/CEIT_Forms/<Course_Sec>_<Sched>_SYLLABUS_ACCEPTANCE.docx` | Raises `TemplateError`, `TypeError`. Caught by orchestrator; logged to `results["errors"]["ceit"]` | `test_generator_factory_loads_all_7` (factory loading/template existence only). *(Note: `test_generic_doc_gen_syllabus` exercises generic `ConfigurableDocumentGenerator`, not `SyllabusGenerator`)* | **Source-verified**<br>**Factory-registration test-verified**<br>**Template-verified**<br>*(Successful native generation not directly evidenced by cited tests)* |
| **2** | **ExamReturnsGenerator (Midterm)** | `ExamReturnsGenerator.generate()` | `.docx` | `template_exam_midterm.docx`<br>Profile: `exam_returns` (`academic_docx`) | `ClassInfo`, `period="MIDTERM"` | `<Output>/<Course_Sec>/CEIT_Forms/<Course_Sec>_<Sched>_EXAM_RETURNS_MIDTERM.docx` | Raises `TemplateError`, `TypeError`. Caught by orchestrator; logged to `results["errors"]["ceit"]` | `test_generator_factory_loads_all_7` (factory loading/template existence only), `test_template_mutations.py::test_m4_exam_swap_roster_columns` (mutated fixture). *(Note: `test_generic_doc_gen_exam` exercises generic `ConfigurableDocumentGenerator`, not `ExamReturnsGenerator`)* | **Source-verified**<br>**Factory-registration test-verified**<br>**Template-verified**<br>*(Successful native generation not directly evidenced by cited tests)* |
| **3** | **ExamReturnsGenerator (Finals)** | `ExamReturnsGenerator.generate()` | `.docx` | `template_exam_finals.docx`<br>Profile: `exam_returns` (`academic_docx`) | `ClassInfo`, `period="FINAL"` | `<Output>/<Course_Sec>/CEIT_Forms/<Course_Sec>_<Sched>_EXAM_RETURNS_FINALS.docx` | Raises `TemplateError`, `TypeError`. Caught by orchestrator; logged to `results["errors"]["ceit"]` | `test_generator_factory_loads_all_7` (factory loading/template existence only) | **Source-verified**<br>**Factory-registration test-verified**<br>**Template-verified**<br>*(Successful native generation not directly evidenced by cited tests)* |
| **4** | **TOSGenerator (Midterm)** | `TOSGenerator.generate()` | `.docx` | `template_tos_midterm.docx`<br>Profile: `tos` (`academic_docx`) | `ClassInfo`, `period="Midterm"` (appends to semester_ay) | `<Output>/<Course_Sec>/CEIT_Forms/<Course_Sec>_<Sched>_TOS_MIDTERM.docx` | Raises `TemplateError`, `TypeError`. Caught by orchestrator; logged to `results["errors"]["ceit"]` | `test_generator_factory_loads_all_7` (factory loading/template existence only) | **Source-verified**<br>**Factory-registration test-verified**<br>**Template-verified**<br>*(Successful native generation not directly evidenced by cited tests)* |
| **5** | **TOSGenerator (Finals)** | `TOSGenerator.generate()` | `.docx` | `template_tos_finals.docx`<br>Profile: `tos` (`academic_docx`) | `ClassInfo`, `period="Finals"` (appends to semester_ay) | `<Output>/<Course_Sec>/CEIT_Forms/<Course_Sec>_<Sched>_TOS_FINALS.docx` | Raises `TemplateError`, `TypeError`. Caught by orchestrator; logged to `results["errors"]["ceit"]` | `test_generator_factory_loads_all_7` (factory loading/template existence only) | **Source-verified**<br>**Factory-registration test-verified**<br>**Template-verified**<br>*(Successful native generation not directly evidenced by cited tests)* |
| **6** | **GradeDiscussionGenerator (Midterm)** | `GradeDiscussionGenerator.generate()` | `.docx` | `Midterm-Grade-Discussion_LATEST.docx`<br>Profile: `grade_discussion` (`academic_docx`) | `ClassInfo`, `period="Midterm"` | `<Output>/<Course_Sec>/CEIT_Forms/<Course_Sec>_<Sched>_GRADE_DISCUSSION_MIDTERM.docx` | Raises `TemplateError`, `TypeError`. Caught by orchestrator; logged to `results["errors"]["ceit"]` | `test_generate_midterm_grade_discussion`, `test_factory_includes_grade_discussion` | **Source-verified**<br>**Test-verified**<br>**Template-verified** |
| **7** | **GradeDiscussionGenerator (Finals)** | `GradeDiscussionGenerator.generate()` | `.docx` | `Final-Grade-Discussion_LATEST.docx`<br>Profile: `grade_discussion` (`academic_docx`) | `ClassInfo`, `period="Finals"` | `<Output>/<Course_Sec>/CEIT_Forms/<Course_Sec>_<Sched>_GRADE_DISCUSSION_FINALS.docx` | Raises `TemplateError`, `TypeError`. Caught by orchestrator; logged to `results["errors"]["ceit"]` | `test_generate_finals_grade_discussion`, `test_grade_discussion_finals_formatting_parity` | **Source-verified**<br>**Test-verified**<br>**Template-verified** |
| **8** | **Attendance (Lecture Only)** | `generate_attendance_for_month()` | `.docx` | `attendance/template lec.docx`<br>Direct dynamic column builder | `info: dict`, `students: list`, `month: str`, `year: str`, `class_day: str`, bounds | `<Output>/<Course_Sec>/Attendance/<Course_Sec>_<Sched>_ATTENDANCE_<Day>_<Month>.docx` | Returns `"skipped_empty"` on `EmptyDateError`. Other exceptions caught by orchestrator; logged to `results["errors"]["attendance"]` | `test_attendance_schedule_meetings_and_dates`, `test_attendance_default_template`, `test_attendance_name_scaling` | **Source-verified**<br>**Test-verified**<br>**Template-verified** |
| **9** | **Attendance (Lecture + Lab)** | `generate_attendance_for_month()` | `.docx` | `attendance/template lab and lec.docx`<br>Direct dynamic column builder | `info: dict`, `students: list`, `month: str`, `year: str`, `class_day: str`, bounds | `<Output>/<Course_Sec>/Attendance/<Course_Sec>_<Sched>_ATTENDANCE_<Day>_<Month>.docx` | Returns `"skipped_empty"` on `EmptyDateError`. Other exceptions caught by orchestrator; logged to `results["errors"]["attendance"]` | `test_attendance_schedule_parsing`, `test_parse_explicit_schedule_meetings_handles_day_slot_pairs` | **Source-verified**<br>**Test-verified**<br>**Template-verified** |
| **10** | **GradeGenerator (Lecture Only)** | `GradeGenerator.generate()` | `.xlsx` | `GRADING_LECTURE_TEMPLATE.xlsx`<br>Profile: `grade_sheet_xlsx` (2 sheets) | `info: dict`, `students: list`, `output_path: str` | `<Output>/<Course_Sec>/Grades/<Course_Sec>_<Sched>_GRADING_SHEET.xlsx` | Cleans up `.tmp.xlsx`. Raises `TemplateError`, `ValueError`. Caught by orchestrator; logged to `results["errors"]["grades"]` | `test_generate_lecture_only`, `test_xlsx_template_contract`, `test_template_mutations` | **Source-verified**<br>**Test-verified**<br>**Template-verified** |
| **11** | **GradeGenerator (Lecture + Lab)** | `GradeGenerator.generate()` | `.xlsx` | `GRADING_LECTURE_LAB_TEMPLATE.xlsx`<br>Profile: `grade_sheet_xlsx` (4 sheets) | `info: dict`, `students: list`, `output_path: str` | `<Output>/<Course_Sec>/Grades/<Course_Sec>_<Sched>_GRADING_SHEET.xlsx` | Cleans up `.tmp.xlsx`. Raises `TemplateError`, `ValueError`. Caught by orchestrator; logged to `results["errors"]["grades"]` | `GradeGeneratorTests.setUpClass`, `test_xlsx_template_contract`, `test_grade_gen_normalization` | **Source-verified**<br>**Test-verified**<br>**Template-verified** |
| **12** | **ConfigurableDocumentGenerator** | `ConfigurableDocumentGenerator.generate()` | `.docx` | Custom `.docx` template path<br>Profile: `custom_docx` | `ClassInfo`, `recipe: ValidatedTemplateRecipe` or `dict` | `<Output>/<Course_Sec>/CEIT_Forms/<Course_Sec>_<Sched>_<CUSTOM_SUFFIX>.docx` | Custom template loading/validation failures caught/logged by `GeneratorFactory` (batch continues). Document generation failures caught/logged by orchestrator `results["errors"]["ceit"]` | `test_generic_doc_gen_syllabus`, `test_generic_doc_gen_exam`, `test_custom_template_pipeline.py::test_generator_factory_includes_custom_templates` | **Source-verified**<br>**Test-verified**<br>**Template-verified** |

---

## ⚙️ Execution Strategy in Orchestrator (`process_all`)

When `process_all` runs for a given class section:

1. **Class and Schedule Matching**: Identifies matching timetable blocks from `parsed_schedules` corresponding to the student roster schedule code.
2. **Laboratory Classification**:
   - `type_overrides` UI configuration.
   - Schedule block `type == "LAB"`.
   - Subject title contains `"LAB"` or `"LABORATORY"`.
   - `KNOWN_LAB_SUBJECT_CODES` registry match (`is_known_lab_subject(subject_name)`).
3. **CEIT Forms Execution**:
   - Loops over callables from `GeneratorFactory(templates_dir).get_all()`.
   - Renders 7 native Word documents (`_SYLLABUS_ACCEPTANCE`, `_EXAM_RETURNS_MIDTERM`, `_EXAM_RETURNS_FINALS`, `_TOS_MIDTERM`, `_TOS_FINALS`, `_GRADE_DISCUSSION_MIDTERM`, `_GRADE_DISCUSSION_FINALS`).
   - Renders any registered and enabled Custom Templates via `ConfigurableDocumentGenerator`.
   - Writes outputs to `<Output>/<Course_Sec>/CEIT_Forms/`.
4. **Attendance Sheets Execution**:
   - Resolves template: `attendance/template lab and lec.docx` if `has_lab` is true, otherwise `attendance/template lec.docx`.
   - Iterates across semester months (e.g. `[2, 3, 4, 5, 6]` for 2nd semester; `[8, 9, 10, 11, 12]` for 1st semester, or bounded by UI `date_overrides`).
   - Calls `generate_attendance_for_month()`. If `"skipped_empty"`, logs to `results["skipped"]["attendance"]`; if `"generated"`, logs to `results["generated"]["attendance"]`.
   - Writes outputs to `<Output>/<Course_Sec>/Attendance/`.
5. **Grading Sheet Execution**:
   - Resolves template: `GRADING_LECTURE_LAB_TEMPLATE.xlsx` if `has_lab` is true, otherwise `GRADING_LECTURE_TEMPLATE.xlsx`.
   - Resolves validated recipe using `TemplateRecipeResolver.get_instance().resolve(template_path, "grade_sheet_xlsx")`.
   - Instantiates `GradeGenerator(template_path, recipe)`.
   - Executes `grade_gen.generate(grade_info, students, grade_out_path)` which preserves formula tabs and clamps students to validated recipe capacity.
   - Writes output to `<Output>/<Course_Sec>/Grades/`.
