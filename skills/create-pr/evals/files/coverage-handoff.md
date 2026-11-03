# Fixture: Coverage Handoff

## Request and repository rules

Draft a title and description only. Use `fix(ci)`. There is no required PR template.

## Complete change evidence

- The PR changes 17 files across workflow wiring, reporter configuration, and caller guidance.
- The analysis job currently reruns tests even though the test job already produced coverage.
- A repeated fixture setup can fail independently of the successful test job.
- The test job now uploads its LCOV report as `test-coverage`.
- The analysis job downloads that artifact instead of running tests again.
- Reporter configuration writes source paths relative to the repository root.
- Caller guidance explains how to upload the report and pass the artifact name.
- Callers must provide an artifact containing `lcov.info` before enabling analysis.
- No performance measurements or completed validation results are available.

Use only this supplied evidence. No live repository or GitHub access is needed or authorized.
