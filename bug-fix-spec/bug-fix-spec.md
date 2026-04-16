# Bugfix Spec: [Title / Issue ID]

## Current Behavior
WHEN [specific trigger condition with exact details]
THEN [observed incorrect behavior]
- Error: [exact error message or stack trace]
- Environment: [OS, browser, version, deploy SHA]
- Frequency: always | intermittent | under specific load

## Expected Behavior
WHEN [same trigger condition]
THEN [correct behavior, referencing original feature spec if available]

## Unchanged Behavior
WHEN [related but unaffected scenarios — list 2-3 explicitly]
THEN system SHALL CONTINUE TO behave as before
- [Adjacent feature 1 must not regress]
- [Data flow X must remain intact]

## Fix Constraints
- Maximum change scope: [specific files or modules]
- Do NOT modify: [explicit list]
- Do NOT add new dependencies

## Verification
- [ ] Failing test reproduces the bug before fix
- [ ] Fix makes the test pass
- [ ] All existing tests still pass
- [ ] Regression test added for this specific case