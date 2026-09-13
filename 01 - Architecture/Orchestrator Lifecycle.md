---
title: "Orchestrator Lifecycle"
tags:
  - cvsu-generator
  - architecture
  - orchestrator
  - concurrency
status: active
last_modified: 2026-09-13
source_of_truth:
  - modules/services/orchestrator.py
  - executable_test/api/generation.py
---

# Orchestrator Lifecycle

The **Orchestrator** (`modules/services/orchestrator.py`) is the centralized processing coordinator. It consumes parsed schedule and roster data, drives all document generation engines, tracks real-time progress, handles cancellation, and compiles structured results.

Related notes:
- [[CvSU Document Generator MOC]]
- [[System Architecture]]
- [[PyWebView Bridge]]
- [[Generators Overview]]

---

## ⚙️ Core Service: `process_all`

The primary entry point is `process_all(...)` in `modules/services/orchestrator.py`:

```python
def process_all(
    schedule_path,
    xlsx_files,
    output_dir_base,
    type_overrides=None,
    date_overrides=None,
    class_filter=None,
    engine_filter=None,
    progress_callback=None,
    cancel_event=None,
    roster_configs=None,
)
```

| Parameter | Type | Purpose |
|---|---|---|
| `schedule_path` | `str` | Path to master schedule `.xls(x)` |
| `xlsx_files` | `list[str]` | Roster file paths |
| `output_dir_base` | `str` | Root output directory |
| `type_overrides` | `dict` | `{class_id: "lecture_lab" \| "lecture"}` manual lab-type overrides |
| `date_overrides` | `dict` | `{startMonth, startDay, endMonth, endDay, startYear}` |
| `class_filter` | `set` | Restrict to specific class IDs, schedule codes, or sections |
| `engine_filter` | `set` | Restrict to `{"attendance", "ceit", "grades"}` subset |
| `progress_callback` | `callable` | Receives telemetry dict after every completed document |
| `cancel_event` | `threading.Event` | Polled at every iteration boundary |
| `roster_configs` | `dict` | Per-file `{name_col, id_col, header_row, linked_schedule_code, ...}` |

---

## 🔄 Two-Pass Architecture

`process_all` uses an explicit **two-pass design** to produce smooth, accurate progress tracking:

### Pass 1 — Pre-calculation (dry run over all roster files)

The function iterates all `xlsx_files` _before_ generating any documents to compute the exact total step count:

```python
num_ceit_generators = len(factory.get_all()) if "ceit" in enabled_engines else 0
total_estimated_steps = 0
for student_file in xlsx_files:
    # Parse filename, find schedule blocks, compute attendance months
    total_estimated_steps += num_ceit_generators       # one per CEIT form
    total_estimated_steps += len(grouped) * len(months) # per subject × month
    total_estimated_steps += 1                          # grading sheet
total_estimated_steps = max(1, total_estimated_steps)
```

This guarantees smooth linear progress without step-count guessing. The value is dynamic — it reflects the actual registered `GeneratorFactory.get_all()` length at runtime, including any custom templates.

### Pass 2 — Generation (actual document writes)

The second pass re-iterates the same files and executes generation. After each document is written:

```python
def _notify(current_class, current_task):
    current_step += 1
    pct = int(min(99, (current_step / total_estimated_steps) * 100))
    progress_callback({
        "percent": pct,
        "current_class": current_class,
        "current_task": current_task,
        "step": current_step,
        "total_steps": total_estimated_steps
    })
```

Progress is capped at 99% until `window.onGenerationComplete` fires, ensuring the UI never shows 100% before the completion callback.

---

## 🗂️ Roster Matching Logic

For each roster file, the orchestrator attempts two matching strategies in order:

### Strategy 1 — Filename Regex Pattern
```
^(.*?)\s*list of students for\s+(\d+)\s*-\s*(.+?)\.(xlsx|xls|csv)
```
Extracts `course_section`, `schedule_code`, and `subject_name` from the filename.

Normalizes program prefixes: strips `BS` prefix, maps `CSCS` → `CS`, reconstructs `{letters}{year}-{section}` format (e.g., `CS1-4`).

### Strategy 2 — Linked Schedule Code (Manual Mapping)
If filename does not match, reads `roster_configs[file]["linked_schedule_code"]` and looks up class details from parsed schedule data via `_find_class_details_by_schedule_code`.

---

## 🔬 Lab Detection Priority Chain

For each matched class, the lab classification follows a strict priority:

1. **User override**: `type_overrides.get(f"{schedule_code}_{course_sec}")` or by schedule code or section alone.
2. **Automatic detection**: any schedule block with `block["type"].upper() == "LAB"`.
3. **Subject name heuristic**: `"LAB"` or `"LABORATORY"` appears in `subject_name.upper()`.
4. **Known lab subject registry**: `is_known_lab_subject(subject_name)` or `base_code in KNOWN_LAB_SUBJECT_CODES`.

If a lab component is detected:
- `attendance/template lab and lec.docx` is used for attendance sheets.
- `GRADING_LECTURE_LAB_TEMPLATE.xlsx` is used for grading sheets.

---

## 📅 Semester Month Resolution

Month ranges are derived from the semester string in the schedule:

| Semester | Default Months |
|---|---|
| First Semester (contains "first" or "1st") | August → December: `[8, 9, 10, 11, 12]` |
| Second Semester (contains "second" or "2nd") | February → June: `[2, 3, 4, 5, 6]` |

**Year resolution for second semester:**
- If two 4-digit years are found in the semester string and semester is second, uses `years[1]` (e.g., `2026-2027` → `2027`).
- Otherwise uses `years[0]`.

**Date override priority:**
If `date_overrides` contains `startMonth`, `startDay`, `endMonth`, `endDay`, the full custom month range supersedes the semester defaults. `start_bound` and `end_bound` tuples are passed to `generate_attendance_for_month` to clip within-month dates.

**Year-wrap handling:**
If the date range spans December → January (e.g., December 2026 → February 2027), months after December receive `att_year + 1`:
```python
year_for_month = (
    att_year + 1
    if (start_bound and end_bound and start_bound[0] > end_bound[0] and m < start_bound[0])
    else att_year
)
```

---

## 🚫 Graceful Cancellation Protocol

The `cancel_event` (`threading.Event`) is checked at **every loop boundary**:
- Start of each roster file iteration.
- After each CEIT form is generated.
- After each attendance month batch.
- After the grading sheet step.

On detection:
```python
if cancel_event and cancel_event.is_set():
    results["cancelled"] = True
    break
```

All files successfully written before the cancel event are preserved. No partially written output files are left open — `save_docx` uses an atomic temp-file-then-rename strategy.

---

## 📊 Result Structure

`process_all` returns a structured dict:

```python
{
    "generated": {
        "attendance": ["CS1-4_202612040_ATTENDANCE_Mon_September.docx", ...],
        "grades":     ["CS1-4_202612040_GRADING_SHEET.xlsx", ...],
        "ceit":       ["CS1-4_202612040_SYLLABUS_ACCEPTANCE.docx", ...]
    },
    "skipped": {
        "attendance": ["... (0 days)", ...],
        "grades":     [],
        "ceit":       [],
        "rosters":    ["unknown_file.xlsx", ...]
    },
    "errors": {
        "attendance": ["Failed Attendance (...): <msg>", ...],
        "grades":     [],
        "ceit":       [],
        "rosters":    []
    },
    "by_class": {
        "CS1-4": {
            "ceit": [{"name": "...", "path": "..."}],
            "attendance": [...],
            "grades": [...]
        }
    },
    "cancelled": False
}
```

---

## 🗂️ Output Directory Structure

For each processed class section, the orchestrator creates:

```text
<output_dir_base>/
└── <course_sec_safe>/
    ├── CEIT_Forms/
    │   ├── <cs>_<sched>_SYLLABUS_ACCEPTANCE.docx
    │   ├── <cs>_<sched>_EXAM_RETURNS_MIDTERM.docx
    │   └── ... (7 total)
    ├── Attendance/
    │   └── <cs>_<sched>_ATTENDANCE_<Days>_<Month>.docx
    └── Grades/
        └── <cs>_<sched>_GRADING_SHEET.xlsx
```

Both `course_sec_safe` and `schedule_code_safe` are sanitized via `sanitize_filename()` before use in filesystem paths.
