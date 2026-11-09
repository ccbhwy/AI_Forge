---
name: test-case-effort-analyzer
description: analyze atm or jira test cases from xml or excel inside vscode, inspect the current automation repository for reusable tests and methods, and estimate playwright script creation level, duration, evidence, and reasons. use when the user asks to assess automation implementation effort for one or more test cases.
---
# Test Case Effort Analyzer

## Purpose
Analyze test cases from XML or Excel and use the open VS Code workspace as evidence to estimate automation script creation effort. Output results only in the VS Code agent conversation. Do not create or modify Excel unless explicitly requested.

## Required workflow
1. Discover the input XML/XLSX/XLS/CSV file supplied by the user or present in the workspace.
2. Parse every requested test case. For XML, support the ATM exporter structure described in `references/atm-schema.md`. For Excel, identify semantically equivalent columns even when names are English, Chinese, or Japanese.
3. Normalize CDATA, HTML tags, HTML entities, lists, and line breaks without losing step boundaries.
4. Extract case name, case key, objective, preconditions, component, folder, labels, steps, and expected results.
5. Count executable test steps. Do not count headings, empty rows, or purely descriptive notes as steps.
6. Inspect the open automation repository before assigning the final level. Search tests, page objects, fixtures, helpers, utilities, components, and shared data.
7. Find evidence at three levels:
   - Similar existing automated cases.
   - Reusable page-object or helper methods.
   - Missing behaviors or methods that must be newly implemented.
8. Estimate reuse coverage as: reusable required behaviors / total required behaviors. Count behaviors, not source lines.
9. Identify complex logic or complex methods using `references/level-rules.md`.
10. Apply the highest matching level. Complexity always overrides a lower step-count level.
11. Return the standard conversational report below. Cite repository evidence with workspace-relative file paths and symbols, and line numbers when available.

## Repository search rules
Search the whole workspace, prioritizing case key, case name, component nouns, UI labels, expected-result phrases, selectors, and action verbs. Include common paths such as `tests/`, `pages/`, `page-objects/`, `fixtures/`, `helpers/`, `utils/`, and `components/`. Exclude `.git/`, `node_modules/`, `dist/`, `build/`, `coverage/`, `playwright-report/`, `test-results/`, and generated artifacts.

Do not claim that a method is reusable based only on a similar name. Open the implementation and verify that parameters, navigation state, locator scope, waiting strategy, and assertions fit the target case.

## Final output format
Start every result with this exact copy-friendly title format:

`<Test case name> (<Case key>)`

Example: `Add order note (FLX-T1300)`

Then output:

### Assessment
- **Level:** Level N
- **Estimated creation time:** N day(s)
- **Step count:** N
- **Complex logic/methods:** N
- **Reuse ratio:** N%
- **Confidence:** High / Medium / Low

### Detailed reason
Provide 3-7 concise numbered reasons. Explain the decisive rule, step count, complexity, reuse evidence, missing implementation, and important risks. Do not output only a generic conclusion.

### Reusable implementation evidence
- `<relative/path>: <symbol or case>` - explain exactly what can be reused.
If none is verified, write `No directly reusable implementation was verified.`

### New implementation required
- List each new page-object method, helper, fixture, data setup, assertion, or synchronization behavior likely required.

### Case summary
List preconditions and each normalized step paired with its expected result in compact form.

### Decision note
State which rule determined the final level and why lower levels were rejected.

## Multiple cases
For multiple cases, preserve input order and repeat the complete format for each case. End with a short summary showing counts by Level and total estimated working days. Never merge distinct cases into one estimate.

## Evidence and uncertainty
- Never invent repository files, methods, selectors, or coverage.
- If the repository is unavailable, explicitly label the result `Provisional` and explain that reuse and complexity cannot be finalized.
- If a case step is ambiguous, state the specific ambiguity and use the more conservative level only when the ambiguity could materially increase implementation work.
- Treat the day estimate as script creation effort under the team's level standard, not elapsed calendar time or execution time.
