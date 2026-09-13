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
