# Fixture: Runtime Dependency Security Update

## Request and repository rules

Draft a title and description only. Choose the Conventional Commit type from the release policy.
The valid scope is `client`. The policy publishes runtime dependency security corrections in a
patch release, new consumer features in a minor release, and development-only maintenance without
a package release. There is no required PR template.

## Complete change evidence

- The PR updates a runtime URL parser within its compatible dependency range.
- The previous version can consume excessive CPU while parsing a malformed untrusted URL.
- The application accepts URLs from external feed entries, so it reaches the affected parser.
- The fixed dependency rejects that input. This is not remote code execution.
- The package's public API and supported runtime versions do not change.
- The latest commit is `chore(deps): update url parser`.
- No validation results, advisory totals, or GitHub alert counts are supplied.

Use only this supplied evidence. No live repository or GitHub access is needed or authorized.
