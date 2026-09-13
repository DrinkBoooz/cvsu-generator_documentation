---
title: "Attendance Templates"
tags:
  - cvsu-generator
  - templates
  - attendance
  - docx
status: active
last_modified: 2026-09-13
source_of_truth:
  - attendance/
  - modules/generators/attendance_gen.py
---

# Attendance Templates

Specifications for Microsoft Word (`.docx`) attendance templates located in `attendance/`.

Related notes:
- [[CvSU Document Generator MOC]]
- [[Template Guidelines]]
- [[Attendance Generator]]

---

## 📅 Template Layout & Tables

The attendance template consists of two primary sections:

1. **Header Block**:
   - Institutional CvSU header with logo and college title.
   - Metadata rows:
     - Course & Section
     - Schedule Code
     - Subject Description
     - Meeting Day & Time (e.g. `Monday 7:00 AM - 10:00 AM`)
     - Room Assignment
     - Month & Year (e.g. `September 2026`)

2. **Attendance Grid**:
   - **Column 1**: Row Number (1 to $N$)
   - **Column 2**: Student Name (Alphabetical, Surname first)
   - **Columns 3 to 7+**: Calendar Meeting Dates. The generator dynamically populates the column headers with the exact numeric days of that month (e.g., `4`, `11`, `18`, `25`).
   - **Summary Columns**: Total Present, Total Absent, Remarks.
