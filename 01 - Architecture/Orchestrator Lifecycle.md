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

The **Orchestrator** (`modules/services/orchestrator.py`) is the centralized operational coordinator. It drives document generation across all selected classes and engines, tracks progress, handles thread interruption, and compiles execution results.

Related notes:
- [[CvSU Document Generator MOC]]
- [[System Architecture]]
- [[PyWebView Bridge]]
- [[Generators Overview]]

---

## ⚙️ Core Service: `process_all`

The primary entry point is:
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

### 1. Dynamic Total Step Calculation
The orchestrator dynamically calculates the total number of progress steps before initiating processing:
```python
factory = GeneratorFactory(templates_dir)
num_ceit_generators = len(factory.get_all())
total_steps = num_classes * (num_ceit_generators + 1 + num_months)
```
This guarantees smooth, linear progress bar movements without hardcoding step counts.

### 2. Live Progress Telemetry
After every individual document is written to disk, the orchestrator invokes `progress_callback`:
```python
progress_callback({
    "current_step": step_count,
    "total_steps": total_steps,
    "percent": round((step_count / total_steps) * 100, 1),
    "message": f"Generated {doc_name} for {section}",
    "class_name": section,
    "engine": engine_name
})
```

### 3. Graceful Cancellation Protocol
At the start of every class iteration and before each document generation step, the engine checks:
```python
if cancel_event and cancel_event.is_set():
    logger.info("Cancellation event detected. Halting generation loop.")
    results["cancelled"] = True
    return results
```
- Halting is instantaneous and does not leave partially written file handles open.
- Output files that have already completed are preserved cleanly.

### 4. Telemetry Result Structure
`process_all` returns a structured report dictionary:
```python
{
    "generated": {"attendance": [...], "grades": [...], "ceit": [...]},
    "skipped": {"attendance": [...], "grades": [...], "ceit": [...], "rosters": [...]},
    "errors": {"attendance": [...], "grades": [...], "ceit": [...], "rosters": [...]},
    "by_class": {
        "BSCS 1-4": {"status": "complete", "count": 10}
    },
    "cancelled": False
}
```
