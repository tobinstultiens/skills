---
name: plm-tralin-annotations
description: >-
    Skill for adding or reconciling Tralin annotations in the PLM application based on a selected
    PLM RSD branch. Use when the user asks which Tralin annotations should be added, updated, or
    checked against an RSD branch.
user-invocable: true
---

# PLM Tralin annotation reconciliation

Use this skill when working in `plm-application` and the task is to add, refresh, or validate Tralin source-code annotations based on changes in `plm-application-rsd`.

## Core lessons learned

- Treat annotation work as traceability work, not feature work, unless the user explicitly asks to implement missing functionality.
- If the user says annotations should be added to "changed parts of this branch", limit edits to the application branch diff. Do not annotate unrelated existing code just because the RSD branch has a matching requirement.
- Compare two independent diffs:
  - RSD branch vs `master`: identifies new/changed/deleted work items.
  - application branch vs `master`: identifies source files and hunks eligible for annotation changes.
- Intersect those diffs before editing. Candidate IDs outside changed application hunks should be deferred, not added.
- Add Tralin annotations only where the code or test directly implements or verifies the RSD statement. If labels, sorting, column order, colors, or behavior do not exactly match the RSD text, do not annotate that ID.
- Avoid global annotation churn. Do not run the RSD helper script in write mode against the full app repository unless the user explicitly wants global refresh behavior.

## Preferred workflow

1. Confirm the RSD repo branch and the application repo branch:

   ```powershell
   git --no-pager -C C:\Projects\Documents\plm-application-rsd branch --show-current
   git --no-pager -C C:\Projects\Repos\plm-application branch --show-current
   ```

2. Inspect the RSD branch diff:

   ```powershell
   git --no-pager -C C:\Projects\Documents\plm-application-rsd diff --name-status master...HEAD
   ```

3. Inspect the app branch diff:

   ```powershell
   git --no-pager -C C:\Projects\Repos\plm-application diff --name-status master...HEAD
   git --no-pager -C C:\Projects\Repos\plm-application diff --stat master...HEAD
   ```

4. Parse changed RSD work-item IDs and scan current source annotations. The RSD helper script can be imported for analysis, but do not call its `main`/write path unless global updates are desired:

   ```powershell
   Set-Location C:\Projects\Documents\plm-application-rsd
   python .\scripts\updateSourceCodeAnnotations.py C:\Projects\Repos\plm-application
   ```

   The command above writes to files; prefer importing the script functions from a small analysis snippet when only creating a checklist.

5. Build a candidate checklist:

   - `add`: RSD item is new/changed and the app branch changed hunk directly implements it.
   - `refresh`: an existing annotation in a changed app hunk needs current RSD text.
   - `test-only`: a changed test directly verifies the RSD item.
   - `defer`: outside app branch diff, not implemented, or behavior does not exactly match.

6. Mark only branch-limited annotations with `!` before running Tralin:

   ```text
   {{!C00033}}
   ```

7. Run Tralin from the application package directory using the local executable and the selected RSD branch:

   ```powershell
   Set-Location C:\Projects\Repos\plm-application\PlmApplication
   .\Tralin.exe update RSD@<rsd-branch>
   ```

   Example:

   ```powershell
   .\Tralin.exe update RSD@RD-9284
   ```

   Do not rely on `npm run tralin-update` unless `Tralin.exe` is on `PATH`. In this environment, the executable is stored at:

   ```text
   C:\Projects\Repos\plm-application\PlmApplication\Tralin.exe
   ```

8. If Tralin was accidentally run without `RSD@<branch>`, it may pull `master` and replace new RSD branch IDs with `TODO: Reference to unknown work item`. Fix by re-marking only the branch-limited annotations with `!` and rerunning:

   ```powershell
   .\Tralin.exe update RSD@<rsd-branch>
   ```

9. After Tralin runs, verify:

   ```powershell
   rg "\{\{![A-Z][0-9]{5}\}\}|TODO: Reference to unknown work item" C:\Projects\Repos\plm-application\PlmApplication
   git --no-pager -C C:\Projects\Repos\plm-application diff --stat
   git --no-pager -C C:\Projects\Repos\plm-application diff --check
   npm --prefix C:\Projects\Repos\plm-application\PlmApplication run elm
   ```

10. Review the final diff to confirm edits are limited to changed branch files/hunks. If Tralin touched unrelated files because of pre-existing `!` markers, decide whether they are branch-owned changes; otherwise revert only the out-of-scope annotation churn.

## PLM-specific caution points

- Tralin warnings for deleted or malformed old annotations may be pre-existing and out of scope. Do not clean them up unless they are in the changed branch hunks or the user asks for global cleanup.
- `S00213` and `S00216` may appear as deleted/unknown in older shared card/compare code. Do not remove them during a branch-limited task unless those exact hunks are being changed.
- Manufacturer-parts RSD IDs require exact behavior matching:
  - Do not add `C00036` if the UI label is `Part Number` instead of `Manufacturer part number`.
  - Do not add `C00046` if the UI label is `Quantity` instead of `Supply form quantity`.
  - Do not add `C00056` if default sorting is not proven to match the specified order.
  - Do not add `C00057` if default column order does not match the RSD order exactly.
  - Do not add `C00048` if the badge status/color behavior does not fully match the RSD status badge design.

## Recommended final response shape

Include:

1. What was annotated/refreshed.
2. Which RSD branch Tralin was run against.
3. Which validation commands passed.
4. Which candidate IDs were intentionally deferred and why.
