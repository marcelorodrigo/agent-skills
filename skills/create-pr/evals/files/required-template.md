# Fixture: Repository Template Takes Precedence

## Request and repository rules

Draft a title and description only. Use `fix(cache)` for the correction. The repository requires
the following description headings, including validation results in the body:

```markdown
## Summary
## Compatibility
## Validation
```

## Complete change evidence

- The PR invalidates the product cache before publishing an update event.
- Currently, subscribers can receive the event while the previous cache entry remains available.
- The public API and event payload stay the same.
- All 14 local unit tests passed on this diff. CI has not run.
- No other validation or compatibility claims are established.

Use only this supplied evidence. No live repository or GitHub access is needed or authorized.
