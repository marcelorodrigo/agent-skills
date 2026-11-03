# Fixture: Follow-Up Fixes in a Feature PR

## Request and repository rules

Revise the existing PR title and description, locally only. The repository uses Conventional
Commits with scope `i18n`. Adding a new library is a feature. There is no required template.

## Complete change evidence

- The existing PR adds a new locale library that has never been released.
- It provides locale registration, locale switching, and fallback-message resolution.
- Its agreed purpose is adding that library, not upgrading workspace dependencies.
- Follow-up commits fix tests, align dependencies with the existing workspace, and rename an
  option introduced earlier in this same unpublished feature branch.
- The latest commit subject is `fix(i18n): correct locale fallback tests`.
- The current PR title repeats that latest commit subject.
- No published consumer API changes, application migrations, or unrelated dependency updates
  are part of the diff. No completed validation results are supplied.

Use only this supplied evidence. No live repository or GitHub access is needed or authorized.
