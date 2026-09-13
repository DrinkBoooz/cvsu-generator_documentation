---
title: "Attendance Generator"
tags:
  - cvsu-generator
  - generators
  - attendance
  - docx
status: active
last_modified: 2026-09-13
source_of_truth:
  - modules/generators/attendance_gen.py
  - attendance/
---

# Attendance Generator

The **Attendance Generator** (`modules/generators/attendance_gen.py`) builds monthly attendance rosters customized to the meeting days and time schedule of each class section.

Related notes:
- [[CvSU Document Generator MOC]]
- [[Generators Overview]]
- [[Attendance Templates]]
- [[Template Guidelines]]

---

## 📅 Schedule Matching & Calendar Math

1. **Token Parsing**:
   - Parses day strings from schedule blocks: `M`, `T`, `W`, `TH`, `F`, `S`.
   - Maps each day to calendar weekdays (`0 = Monday`, `3 = Thursday`, etc.).
2. **Asynchronous Session Filtering**:
   - Schedule rows containing `Async` or `Asynch` are detected and excluded from the meeting calendar, ensuring attendance logs only track physical or synchronous in-person class dates.
3. **Monthly Iteration**:
   - Computes all calendar occurrences of meeting weekdays in each month of the semester.
   - Populates column date headers (e.g. `Sept 4`, `Sept 11`, `Sept 18`, `Sept 25`).
4. **Semester Boundary Support**:
   - Automatically filters out dates outside the configured `start_date` and `end_date` (or standard institutional semester range).
   - Semester defaults: First Semester (Aug-Dec), Second Semester (Feb-Jun).
   - If the date range spans a year boundary (e.g. Dec to Jan), the calendar math automatically increments the year calculation for the months that fall past December.

---

## 🔄 Orchestration Call Chain

The generator is driven directly by `orchestrator.py` via the `generate_attendance_for_month` function.

### Contract & Signature
```python
def generate_attendance_for_month(
    info: ClassInfo, 
    students: list, 
    month: int, 
    year: int, 
    template_path: str, 
    output_path: str,
    start_bound: tuple = None,
    end_bound: tuple = None
) -> int
```

- **Arguments**: 
  - `info`: The populated `ClassInfo` data class containing the detected schedule days.
  - `students`: A list of `(Name, StudentID)` tuples parsed by the `roster_parser`.
  - `month` / `year`: The specific calendar month/year being generated.
  - `template_path`: Resolved by the orchestrator (either lecture or lecture+lab).
  - `start_bound` / `end_bound`: Optional `(Month, Day)` tuples used to truncate dates that fall outside the semester period.

- **Semester Resolution & Date Overrides**: 
  The month and year are computed dynamically by `orchestrator.py` based on semester strings (e.g. `[8, 9, 10, 11, 12]` for First Semester). If `date_overrides` are provided via the UI, they supersede the semester defaults, passing explicit `start_bound` and `end_bound` truncations.

- **Template Choice**: 
  The orchestrator selects the template based on lab detection. If `auto_has_lab` is true, the `template lab and lec.docx` is passed to `template_path`. Otherwise, `template.docx` is used.

- **Return Structure & Empty Behavior**: 
  The function returns an integer `num_days` representing the number of valid synchronous class meeting days found in that month. 
  - If `num_days == 0` (e.g., month has no matching weekdays within the bounds, or schedule is purely `Async`), the orchestrator captures this and logs it to the `skipped` category, skipping output file creation.
  - If a file lock error occurs during generation (e.g., user has the file open), it relies on `docx_utils` exponential backoff retries, bubbling up to the `errors` category in the orchestrator if exhausted.

---

## 📁 Output Location & Naming

Attendance files are stored under:
```text
<Output Directory>/<Course_Section>/Attendance/
```
Naming pattern:
```text
<Course_Sec>_<SchedCode>_ATTENDANCE_<Day>_<Month>.docx
```
Example:
```text
CS1-4_202612040_ATTENDANCE_Mon_September.docx
```
