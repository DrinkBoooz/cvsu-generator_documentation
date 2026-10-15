---
title: "Attendance Templates"
tags:
  - cvsu-generator
  - templates
  - attendance
  - docx
status: active
last_modified: 2026-10-15
source_of_truth:
  - attendance/
  - modules/generators/attendance_gen.py
  - modules/parsers/calendar_resolver.py
  - modules/models/calendar.py
  - modules/parsers/docx_grid_table.py
  - modules/parsers/template_analyzer.py
  - modules/services/orchestrator.py
---

# Attendance Templates

Specifications for Microsoft Word (`.docx`) attendance templates located in `attendance/`.

Related notes:
- [[CvSU Document Generator MOC]]
- [[Template Guidelines]]
- [[Attendance Generator]]

---

## 📅 Template Layout & Tables

The application utilizes two distinct `.docx` attendance templates stored in the `attendance/` directory, selected dynamically during generation:

### 1. `template lec.docx` (Lecture Only)
Used for standard courses without a laboratory component.
- Features a single row per student and unified schedule metadata.

### 2. `template lab and lec.docx` (Lecture & Laboratory)
Used for courses with integrated laboratory hours.
- Accommodates multi-session class schedules across split lecture and laboratory contact hours.

Both templates consist of two primary tables:

1. **Header Block (Table 0 — Info Table)**:
   - Institutional CvSU header with logo and college title.
   - 5 metadata rows:
     - Row 0: Course Code & Subject Title (Col 1), Month & Year (Col 4).
     - Row 1: Meeting Day & Time Schedule (Col 1).
     - Row 2: Semester and Academic Year (Col 1).
     - Row 3: Room Assignment (Col 1).
     - Row 4: Name of Instructor (Col 1).

2. **Attendance Grid (Table 1)**:
   - **Column 1**: Row Number (`1..N`). Fixed percentage width: `248 pct`.
   - **Column 2**: Student Name (Alphabetical, Surname first). Base percentage width: `1277 pct` + column pool remainder. The generator progressively scales the font size via `_auto_scale_attendance_name`:
     - `<= 24` chars: 8pt (`sz="16"`, bold)
     - `25–28` chars: 7pt (`sz="14"`)
     - `29–33` chars: 6.5pt (`sz="13"`)
     - `34–37` chars: 5.5pt (`sz="11"`, unbold)
     - `>= 38` chars: 5pt (`sz="10"`, unbold)
   - **Column 3**: Student Number. Fixed percentage width: `499 pct`.
   - **Columns 4 to 4+N**: Calendar Meeting Dates. Allocated from the remaining date pool (`5000 - 2577 = 2423 pct`), divided evenly across session count (`DATE_W = 2423 // n_date_cols`).
   - **Summary Columns**: Total Present (`212 pct`), Total Absent (`208 pct`), Remarks (`133 pct`).

---

## 🏛️ Academic Calendar & Holiday Exception Modes

When the attendance generator renders the dynamic date columns (Columns 4 through 4+N), it evaluates each candidate class date against the 3-Layer Calendar Resolver (`modules/parsers/calendar_resolver.py`):

### 1. Matrix Policy Modes
- **`OMIT_COLUMN` (Default)**: Non-working holidays, `#WalangPasok` weather/transport suspensions, and campus events are excluded from the column sequence entirely. Only active class meeting sessions form date columns. This maximizes the physical column width `DATE_W` for active sessions.
- **`STAMP_COLUMN`**: The scheduled date remains in the column sequence in Header Rows 0 and 1, and the student data cells below that date are stamped with an abbreviated status tag:
  - `HOLIDAY` (National & Cavite provincial holidays)
  - `SUSP.` (Typhoon, transport, or institutional class suspensions)
  - `EXAM` (Midterm or Final examination weeks)
  - `EVENT` (University foundation week, sports fest, student convocation)

### 2. OpenXML Cell Border & Font Preservation
In `STAMP_COLUMN` mode, cell styling is rendered via `_format_stamp_cell(tc, tag, date_w)`:
- Preserves template physical border elements (`<w:tcBorders>`) inherited from the prototype student cell.
- Centers text alignment via `<w:jc w:val="center"/>`.
- Applies responsive 4-tier font auto-scaling to prevent cell overflow in multi-session weeks:
  - `DATE_W >= 200`: 8pt (`sz="16"`, bold)
  - `DATE_W >= 150`: 7pt (`sz="14"`, bold)
  - `DATE_W >= 100`: 6pt (`sz="12"`, bold)
  - `DATE_W < 100`: 5pt (`sz="10"`, bold)

---

## 🔬 Merged-Grid Normalization & TemplateManifest Diagnostics

### 1. Complex Header Grid Normalization (`DocxGridTable`)
Attendance templates feature multi-level merged cells in the matrix table:
- **Horizontal Merges (`<w:gridSpan>`)**: Header Row 0 groups multiple meeting days into ISO calendar weeks (`WEEK 1`, `WEEK 2`, ...).
- **Vertical Merges (`<w:vMerge>`)**: Summary columns (`LB`, `LC`, `Remarks`) span Header Rows 0 and 1 vertically, creating continuation cells.
- **Visual Matrix Mapping**: `DocxGridTable` (`modules/parsers/docx_grid_table.py`) flattens these irregular rows into a 2D rectangular grid. Any visual access to merged regions resolves directly to the top-left origin `<w:tc>` element, eliminating python-docx coordinate index distortion.

### 2. In-App Template Diagnostics (`TemplateManifest`)
When an attendance template is inspected via `DeterministicTemplateAnalyzer` (`modules/parsers/template_analyzer.py`), it produces a `TemplateManifest`:
- **Discovered Columns**: Automatically locates `no_col`, `name_col`, `id_col`, and `date_columns_start` without positional guessing.
- **Capacity Measurement**: Measures `template_session_capacity` (number of pre-formatted session columns) and `template_student_row_capacity`.
- **Diagnostics Reporting**: `manifest.get_diagnostics()` verifies presence of the info table (requires $\ge 3$ fields) and matrix structure, emitting actionable alerts if structural columns are missing or malformed.

---

## ⚙️ Template Selection & Lab Detection Logic

The orchestrator dynamically selects between `template lec.docx` and `template lab and lec.docx` based on the `has_lab` resolution.

### Orchestrator Resolution Flow
1. **Explicit Type Override**: If `type_overrides` defines `"lecture_lab"`, the system selects `template lab and lec.docx`.
2. **Schedule Block Type**: If a schedule block `type` explicitly defines `"LAB"`, `has_lab = True`.
3. **Subject Title Heuristics**: If `"LAB"` or `"LABORATORY"` is present in `subject_name`, `has_lab = True`.
4. **Known Lab Subjects Verification**: Cross-references against `KNOWN_LAB_SUBJECT_CODES` (e.g. `DCIT 21`, `COSC 55`, `ITEC 50`) using `is_known_lab_subject(subject_name)`.

Based on this resolved `has_lab` flag, `orchestrator.py` passes `attendance/template lab and lec.docx` or `attendance/template lec.docx` to `generate_attendance_for_month`.

---

## 📁 Output Path Specification
Generated monthly attendance documents are written to:
```text
<Output Directory>/<Course_Section>/Attendance/<Course_Sec>_<SchedCode>_ATTENDANCE_<Day>_<Month>.docx
```
