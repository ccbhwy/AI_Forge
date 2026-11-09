# Script creation level rules

## Precedence
Use the highest level whose conditions are met. Evaluate complex logic first, then step count and reuse. A small step count never lowers a case that contains complex logic.

## Level 0
- Estimated time: 1 working day.
- Effective steps: 3 or fewer.
- No complex logic or complex method.
- Verified reuse ratio is high. Default threshold: 70% or more.

## Level 1
- Estimated time: 2 working days.
- Effective steps: 3 or fewer.
- No complex logic or complex method.
- Reuse ratio is below 70%, or required behavior must be newly implemented but is straightforward.

## Level 2
- Estimated time: 3 working days.
- Effective steps: 4 through 9.
- No complex logic or complex method.

## Level 3
- Estimated time: 4 working days.
- At least one complex logic unit or complex method is required; or
- Effective steps are 10 or more even without complex logic.

## Level 4
- Estimated time: 5 working days.
- Two or more distinct complex logic units or complex methods are required.

## Complex logic or method indicators
Count distinct implementation challenges, not every mention in the test steps. Examples include:
- Multi-window, popup, tab, iframe, canvas, drag-and-drop, or browser permission handling.
- Nontrivial upload/download handling with file-content or format validation.
- Dynamic table/tree/grid targeting requiring compound state-dependent location.
- Branching, loops, parameter combinations, calculation, or cross-field validation.
- Multi-user, multi-role, permission, session, or forced reauthentication flows.
- Database, API, message queue, file system, email, or external-system coordination.
- Long-running asynchronous jobs, polling, retries, uncertain completion signals, or complex synchronization.
- Environment construction, hard-to-create data, cleanup, or destructive state restoration.
- Custom visual, PDF, image, print, or layout comparison beyond ordinary DOM assertions.
- A novel shared method whose implementation itself includes one of the challenges above.

Ordinary navigation, clicking, text entry, simple dropdown selection, standard visible/enabled assertions, and reuse of an already verified helper are not complex by themselves.

## Reuse ratio
Decompose the case into required automation behaviors. Mark a behavior reusable only after verifying an existing implementation can be called or safely adapted. Ratio = verified reusable behaviors / total required behaviors x 100. Similarity alone is evidence for investigation, not proof of reuse.
