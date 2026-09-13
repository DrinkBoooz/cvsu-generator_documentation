---
title: "Input File Conventions"
tags:
  - cvsu-generator
  - guide
  - input-files
status: active
last_modified: 2026-09-13
source_of_truth:
  - modules/parsers/schedule_parser.py
  - modules/parsers/roster_parser.py
  - executable_test/README.md
---

# Input File Conventions

To ensure smooth pairing between your teaching schedule and student rosters, adhere to the file naming and column guidelines below.

Related notes:
- [[CvSU Document Generator MOC]]
- [[User Manual]]
- [[Generator Pipeline]]

---

## 1. Instructor Master Schedule (`.xls` or `.xlsx`)

- **Download Location**: Download your official class timetable schedule directly from the CvSU Faculty Portal.
- **Auto-Extracted Fields**:
  - Faculty Instructor Name
  - College Header (e.g. `College of Engineering and Information Technology`)
  - Semester and Academic Year
  - Course Code, Subject Title, Section, Schedule Code, Days, Times, and Room Assignments.
- **Asynchronous Hours**: Schedule rows containing `Async` or `Asynch` are detected and excluded from the meeting calendar, ensuring attendance logs only track physical or synchronous in-person class dates.

---

## 2. Student Roster Files (`.xlsx`, `.xls`, or `.csv`)

- **Download Location**: Download class student rosters directly from [registrar.cvsu.edu.ph](https://registrar.cvsu.edu.ph/).
- **Required Columns**:
  - **Column A**: Student Full Name (`SURNAME, FIRSTNAME M.I.`)
  - **Column B**: Student Number (`2026XXXXX`)
  - Additional columns (Email, Course, Status, Remarks) are automatically ignored.

### Required Roster File Naming Convention
Roster spreadsheets must match the official portal export format:
```text
{Course/Sec} List of Students for {ScheduleCode}-{Subject}.xlsx
```
**Example**:
```text
BSCS1-4 List of Students for 202612040-DCIT 21A - INTRODUCTION TO COMPUTING.xlsx
```
The program uses `{ScheduleCode}` and `{Course/Sec}` to pair each student list with the corresponding block in your schedule.
