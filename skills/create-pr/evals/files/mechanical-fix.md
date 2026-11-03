# Fixture: Mechanical Configuration Fix

## Request and repository rules

Draft a title and description only. Use Conventional Commits; `fix(ci)` is the valid scope
and type for this correction. There is no required PR template.

## Complete change evidence

- The test runner reads `timezone`; 23 fixture configs currently use the misspelling `timeZone`.
- The PR corrects that key in all 23 configs. There are no other changes.
- The misspelling makes scheduled-job tests use the machine timezone instead of UTC.
- The correct key makes those tests use the UTC value already present in each config.
- No tests or CI runs have been executed for this PR yet.

Use only this supplied evidence. No live repository or GitHub access is needed or authorized.
