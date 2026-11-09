---
name: filter-automated-zephyr-cases
description: Invoke when evaluating uploaded Zephyr XML or Excel exports to identify and exclude cases whose Automation Labels field is exactly Automated before eligibility filtering or scoring.
---

## Purpose
Identify already automated Zephyr cases directly from the uploaded file so they are excluded before scope checks and scoring.

## Uses
Agent capabilities: codeInterpreter

## Instructions
1. Parse the complete uploaded Zephyr XML or Excel export and preserve each case's original ID, name, and fields.
2. Locate the `Automation Labels` field for every case. Match the field name case-insensitively, but treat the value as automated only when its trimmed value equals `Automated` case-insensitively.
3. Normalize IDs for comparison and reporting, treating forms such as `T302` and `FLX-T302` as the same case while displaying `FLX-Txxxx` when possible.
4. Before any eligibility filtering or scoring, exclude every case whose `Automation Labels` value is `Automated`.
5. Do not search agent knowledge sources, repositories, or external exclusion lists to decide whether a case is already automated.
6. Record the total cases inspected, cases excluded by this field, remaining cases, duplicate IDs, and cases where the field is missing, empty, or ambiguous.
7. Include the excluded IDs in a concise audit section and verify that excluded cases receive no score or priority.
8. Validate the count: total unique cases = excluded automated cases + remaining cases. If it does not reconcile, report the discrepancy and stop before claiming a complete result.

## Parameters
- Uploaded Zephyr XML or Excel export.
- Optional user request for concise, detailed, or audit output.

## Output
A normalized exclusion result containing automated case IDs, remaining case IDs, field-quality notes, counts, and reconciliation status for use by the parent agent's evaluation report.