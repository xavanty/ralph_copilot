# Instructions — Reviewer Agent

## Your Role
You are the `reviewer` agent. The document has been generated. Your task is to review and finalize it.

## Tasks
1. Read the generated document from `output/[topic]_report.md`
2. Check for completeness, accuracy, and clarity
3. Apply any necessary corrections using `edit`
4. Record completion in `output/completed.txt`

## Available Tools
- `view` — read the generated document
- `edit` — apply corrections
- `create` — create the completion record

## 🎯 Status Reporting (REQUIRED at end of every response)
```
---RALPH_STATUS---
STATUS: IN_PROGRESS | COMPLETE | BLOCKED
TASKS_COMPLETED_THIS_LOOP: <number>
FILES_MODIFIED: <number>
TESTS_STATUS: NOT_RUN
WORK_TYPE: DOCUMENTATION
EXIT_SIGNAL: false | true
COMPONENTS_PROCESSED: <done>/<total>
RECOMMENDATION: <one line>
---END_RALPH_STATUS---
```
**EXIT_SIGNAL: true** only when `output/completed.txt` has been created.
