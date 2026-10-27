---
title: "Input File Conventions"
tags:
  - cvsu-generator
  - guide
  - input-files
status: active
last_modified: 2026-10-27
source_of_truth:
  - modules/parsers/schedule_parser.py
  - modules/parsers/roster_parser.py
  - modules/parsers/smart_ingest.py
  - modules/models/workspace.py
  - executable_test/README.md
---

# Input File Conventions

To ensure smooth pairing between your teaching schedule and student rosters, adhere to the file naming and column guidelines below.

Related notes:
- [[CvSU Document Generator MOC]]
- [[User Manual]]
- [[Generator Pipeline]]
- [[Application Data Flow]]
- [[Persistent State Architecture]]

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

### Roster File Naming Convention (Auto-Matching)
For automatic pairing, roster spreadsheets should match the official portal export format:
```text
{Course/Sec} List of Students for {ScheduleCode}-{Subject}.xlsx
```
**Example**:
```text
BSCS1-4 List of Students for 202612040-DCIT 21A - INTRODUCTION TO COMPUTING.xlsx
```

### Manual Linking
If you rename your roster files (e.g. `section_1.xlsx`), the program will flag them as "unmatched". You can manually link them to a specific schedule block in the application's Step 2 interface by clicking the "Link Schedule Code" button next to the orphaned file.

---

## 3. Unified Drag-and-Drop "Smart Ingest" Router

Rather than manually navigating multiple file pickers for different document types, the application features a unified **Smart Ingest Classifier** (`SmartIngestClassifier`). You can drag and drop mixed batches of files directly into the window or dropzone:

| File Type / Pattern | Classification | Target Subsystem / Action |
| :--- | :--- | :--- |
| Schedule Table (`.xls`, `.xlsx`, `.csv` with faculty header & time slots) | `schedule` | Parsed into Master Faculty Timetable (Step 1) |
| Class List (`.xlsx`, `.xls`, `.csv` with student names & IDs) | `roster` | Paired with matching schedule section (Step 2) |
| Template Document (`.docx`, `.xlsx` with slot tokens) | `template` | Staged in custom template catalog (Step 3) |
| Academic Calendar (`.ics` iCalendar) | `calendar` | Ingested into Holiday & Academic Schedule Manager |
| Organization Preset (`.cvsu-org.json`) | `org_preset` | Loaded into Campus / Department Configuration |
| Semester Snapshot (`.cvsu-workspace.json`) | `workspace` | Loaded as active Semester Workspace snapshot |

When a batch is dropped, the system classifies each file deterministically and reports an ingestion summary with applied actions and diagnostics.

---

## 4. Mid-Semester Roster Diffing & Updates

When student enrollment changes mid-semester (students adding, dropping, or correcting their names), faculty no longer need to start from scratch or manually verify changes:

1. **Triggering a Diff**:
   - In Step 2, click the **🔄 Diff Update** button on any matched class card, or drag an updated roster spreadsheet over the section card.
2. **Deterministic Comparison Engine**:
   - The engine compares the currently loaded roster against the updated roster file using normalized Student Numbers and names.
   - Changes are categorized into four states:
     - 🟢 **Added**: Newly enrolled students joining the section.
     - 🔴 **Removed**: Students who dropped or withdrew.
     - 🟡 **Modified**: Name corrections or spelling updates for the same student ID.
     - ⚪ **Unchanged**: Students whose enrollment remained identical.
3. **Review & 1-Click Update**:
   - An interactive modal (`#modalRosterDiffBackdrop`) displays breakdown counters and badge-annotated diff cards.
   - Clicking **"Apply Roster Update"** instantly updates the section in-memory and saves changes to persistent state without disturbing other classes.

---

## 5. Named Semester Workspaces (`.cvsu-workspace.json`)

To eliminate re-configuring 15+ settings when switching between semesters or managing multiple faculty profiles, faculty can use **Named Semester Workspaces**:

- **What a Workspace Saves**:
  - Semester name (e.g., `"1st Sem AY 2026-2027"`)
  - Active instructor schedule path
  - All linked class rosters and section configurations
  - Selected output directory
  - Campus / department organization preset
  - Academic calendar configuration
  - Active custom template set
- **1-Click Switching**:
  - The top navigation bar includes a workspace selector (`#selectWorkspace`). Selecting a semester immediately restores the entire workspace state.
- **Updating Active Workspace vs. Saving as New**:
  - **In-Place Update**: When an active semester workspace is loaded, clicking the header **Save** button (`#btnSaveWorkspace`) or clicking **Update Active** in the Workspace Manager modal directly updates and overwrites the active workspace snapshot in-place with zero prompts.
- **Deleting Workspaces**:
  - Workspaces can be deleted via the Workspace Manager modal (`#modalWorkspaceBackdrop`) using either the **🗑️ Delete Active** button (`#btnDeleteActiveWorkspace`) in the active workspace banner or the individual **🗑️ Delete** button next to each saved snapshot in the list.
  - A confirmation modal prevents accidental deletion. Deleting the active workspace safely unloads it and automatically returns the active session to the Default Workspace.
- **Portability & Backup (`.cvsu-workspace.json`)**:
  - Workspaces can be exported as standalone `.cvsu-workspace.json` files via the Workspace Manager (`#modalWorkspaceBackdrop`) and shared or transferred across machines.


