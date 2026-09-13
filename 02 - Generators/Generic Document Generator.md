---
title: "Generic Document Generator"
tags:
  - cvsu-generator
  - generators
  - abstract
status: active
last_modified: 2026-09-13
source_of_truth:
  - modules/generators/ceit_gen.py
---

# Generic Document Generator

This document outlines the abstract base class `DocumentGenerator(ABC)` which acts as the foundation for all `.docx` academic form generators in the application.

Related notes:
- [[CvSU Document Generator MOC]]
- [[CEIT Generator]]
- [[Attendance Generator]]

---

## 📄 `DocumentGenerator` Base Class

Located in `modules/generators/ceit_gen.py`, `DocumentGenerator` is an abstract base class (`ABC`) that enforces a strict interface for rendering Word documents based on CvSU form templates.

### Core Interface
Every generator subclass must implement the following methods:

1. `fill_header(self, body, info: ClassInfo) -> None`
   - Responsible for locating header tables or paragraphs within the document body.
   - Replaces metadata placeholders (e.g. `info.instructor`, `info.course_section`, `info.time_days_room`) using `docx_utils` helper functions.

2. `_fill_student_row(self, cells: list, idx: int, name: str, stnum: str) -> None`
   - Maps student data directly to the corresponding columns of a duplicated table row.
   - For example, `cells[0]` for row number, `cells[1]` for student number, and `cells[2]` for student name.

### Automatic Auto-Scaling
The generic generator architecture mandates the use of `set_cell_text` with font-scaling thresholds (`shrink_threshold`, `shrink_sz`) from `docx_utils.py`. This ensures that unexpectedly long student names or schedule strings do not cause table cells to overflow or wrap awkwardly, maintaining the strict one-page boundaries of the academic forms.

### Registration & Factory Loading
Subclasses of `DocumentGenerator` are registered in the `GeneratorFactory` (e.g. mapping a template file like `"template_consultation.docx"` to a specific generator instantiation lambda). This allows the orchestrator to generically loop over `factory.get_all()` and render all registered forms automatically.
