# ATM exporter XML mapping

Expected structure:
- Project: `/project/projectId`, `/project/projectKey`, `/project/exportDate`
- Test case collection: `/project/testCases/testCase`
- Case ID: `testCase/@id`
- Case key: `testCase/@key`
- Name: `testCase/name`
- Component: `testCase/component`
- Folder: `testCase/folder`
- Objective: `testCase/objective`
- Preconditions: `testCase/precondition`
- Priority: `testCase/priority`
- Status: `testCase/status`
- Automation labels: `testCase/customFields/customField[@name='Automation Labels']/value`
- General labels: `testCase/labels/label`
- Steps: `testCase/testScript[@type='steps']/steps/step`
- Step index: `step/@index`
- Action: `step/description`
- Expected result: `step/expectedResult`

Values may be inside CDATA and contain HTML. Decode entities such as `&nbsp;`; convert `br`, `li`, and paragraph boundaries into meaningful line breaks; remove presentation tags such as `strong`; preserve action-to-expected-result pairing.

If optional fields are absent, continue analysis and mark the field as unavailable. If a file contains multiple test cases, analyze all requested cases or all cases when the user does not specify a subset.
