# Fixture: Related Dependency Updates

## Request and repository rules

Draft a title and description only. Use `chore(ci)`. There is no required PR template.

## Complete change evidence

- Dependabot currently groups unrelated development dependencies together and production
  dependencies together, while splitting related packages across those two groups.
- The PR groups related package families across both dependency types.
- Examples include Nx, Nuxt, and Vitest. Packages that must move together include major updates.
- Remaining patch updates use a separate group.
- Security grouping applies only to root npm jobs and only to packages that must move together.
- Unrelated security fixes remain separate. This does not describe nested manifest jobs.
- GitHub Actions configuration and dependency versions do not change.
- After merge, check that new groups work before replacing existing update PRs.
- The maintenance guide lists exact package patterns and refresh instructions.
- An implementation note says: "Use disjoint pattern-based groups with exact-pinned companions;
  preserve semver-excluded individual-update deferral semantics and open-pull-request limits."
- No tests, security alerts, or hosted updater behavior have been verified for this PR.

Use only this supplied evidence. No live repository or GitHub access is needed or authorized.
