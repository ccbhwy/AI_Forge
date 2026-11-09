---
name: score-zephyr-automation-suitability
description: Invoke after automated-case exclusion and eligibility filtering to distinguish core versus removable physical checks, score each in-scope Zephyr case, and assign the final automation recommendation consistently.
---

## Purpose
Score in-scope Zephyr test cases for Playwright automation suitability. First determine whether any real-device or physical-output check is the case's primary test objective or only a removable secondary checkpoint, then prevent core physical validation from being overrated.

## Instructions
1. Confirm the case passed the parent agent's eligibility filters. Do not score excluded, out-of-scope, low-confidence, or split-required cases.
2. Identify the primary test objective from the name, stated purpose, main expected result, and decisive pass/fail condition. Do not infer it from a device keyword alone.
3. Classify every device or physical-output dependency before scoring:
   - Core physical objective: the case proves printing, scanning, finishing, folding, cutting, binding, color, image quality, paper position, sound, appearance, physical output, or real-device behavior; or it cannot pass without observing the device or produced item. This remains core even when all earlier Web steps are automatable.
   - Removable secondary checkpoint: deleting or separating the physical observation leaves a complete Web objective, executable Web steps, and an objective software-side expected result. The retained case must verify meaningful behavior rather than only a click.
   - Unclear relationship: mark `需人工复核` and do not score.
4. Apply this decision before adding points:
   - A core physical objective is `不适合自动化`; assign no priority. Calculate a traceable score only when the report requires it, using the caps below.
   - For a removable secondary checkpoint, state exactly which steps or expected-result clauses to remove or split and confirm the rewritten objective remains independently testable.
   - Never treat automatable earlier steps as proof that a case with a physical primary pass/fail check is automatable.
5. Build positive points only from explicit evidence in the retained automated scope: clear GUI actions +15; clear API/database/CLI/backend actions +20; ordered executable steps +10; objective deterministic expected results +15; verification through UI text/state, files, system state, API, logs, or database +20; software-preparable preconditions +10; automatically creatable/importable/resettable test data +10.
6. Apply supported deductions and caps: core real printer/scanner/finisher/TU-510/device dependency -50 and cap 59; core physical-output or sensory verification -50 and cap 39; removable secondary physical checkpoint -20 and cap 79; special network -20; complex nonbaseline Web environment -15; multiple machines/servers/environments -15; external services -15 unless a stable local substitute exists; other subjective visual judgment -20; complex or unstable data -10.
7. Avoid double counting. A device name, registration record, queue target, or software configuration value alone does not prove real-device dependency.
8. Calculate 0-100 and then apply caps. Caps override arithmetic.
9. Final judgment: 80-100 `推荐优先自动化` and P1 when confidence is high/medium and difficulty easy/medium; 60-79 `可自动化（有难点）` and P2 when confidence is high/medium; 0-59 `不适合自动化` with no P1/P2/P3; split or unclear cases remain `建议拆分` or `需人工复核` without score or priority.
10. Difficulty is independent and cannot raise priority above the score band.
11. Verify every deduction has source evidence, every core real-device case is below 60, every core physical-output case is at most 39, no sub-60 case is called automatable, and each reduced case names both the removed checkpoint and retained automated assertion.
12. Return primary objective, dependency classification, score, judgment, applicable priority, difficulty, confidence, evidence, deductions, caps, retained scope, removed/split checkpoints, and concise rationale.

## Parameters
- A normalized Zephyr case that passed eligibility filtering.
- Explicit name, folder/component, preconditions, steps, data, expected results, labels, and dependencies.

## Output
A traceable scoring record with the core-versus-secondary physical-check decision, any reduced automated scope, and validation findings.