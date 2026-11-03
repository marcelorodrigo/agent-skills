# Fixture: Action References and Partial Locking

## Request and repository rules

Draft a title and description only. Use `fix(ci)` because the repository requires a patch
release for corrected published workflow behavior. There is no required PR template.

## Complete change evidence

- The user requested supported major action references and a lockfile.
- The diff replaces compatible exact action references with supported major aliases.
- It adds `actions.lock` and verification for four named workflows, not the entire repository.
- Existing internal `./` calls remain outside locking.
- It keeps `pnpm/action-setup@v6.1.0` because the `v6` alias lacks required pnpm 12 support.
- A supporting CMS change binds published sibling actions to the selected release using `$/`.
- Those CMS calls require Actions Runner 2.338.0 or later on GitHub.com or Enterprise Cloud.
- The latest commit is `fix(ci): bind CMS siblings and normalize action dependencies`.
- No major action upgrades or organization enforcement changes are included.
- Validation has not run. The maintenance guide contains per-workflow details.

Use only this supplied evidence. No live repository or GitHub access is needed or authorized.
