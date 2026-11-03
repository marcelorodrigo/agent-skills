---
name: create-pr
description: >-
  Create or update pull requests and draft or revise PR titles and descriptions with clear
  motivation, plain language, evidence, and review risks when preparing changes for review.
license: MIT
compatibility: Requires GitHub CLI (gh) authenticated and available for publishing
metadata:
  version: "1.3.0"
---

# Create Pull Request

Help reviewers understand the purpose, assess correctness, and identify unresolved concerns.

## Prerequisites

- Read applicable repository instructions and PR templates. Follow explicit user and repository
  requirements before this skill's defaults, including validation order and required sections.
- Look for templates in `.github/`, `docs/`, and the repository root, including template
  directories. Select the applicable template.
- Confirm whether the request is writing-only, a new PR, or an update to an existing PR, and
  whether it covers the title, description, or both. Default new PRs to draft unless specified.
- Respect the user's authorization for commits, pushes, and remote PR changes. A request to draft
  text alone does not authorize publication.
- Preserve unrelated user work and published history. Follow the repository's branch-update
  policy; do not automatically rebase, amend, or force-push a shared branch.
- Keep one PR per logical change.

## Process

### Step 1: Confirm the Target and Change Scope

Inspect the repository, current branch, and working tree:

```bash
git status --short --branch
git remote -v
```

For a local writing-only request or a new PR with uncommitted work, inspect intended committed
changes and relevant staged, unstaged, and untracked files. Confirm scope instead of including
unrelated user work:

```bash
git diff --cached
git diff
git ls-files --others --exclude-standard
```

Read the relevant untracked files; listing their names does not reveal their contents.
Use available local base refs for committed changes. Do not require GitHub access for a local
writing-only request; state any comparison limitations if the base is unavailable.

For remote operations, confirm `REPO` as the target `owner/repository`, not necessarily the push
destination. For an existing PR, resolve its number or URL first, inspect its body and base/head
metadata with `gh pr view --repo "$REPO"`, and analyze its diff with `gh pr diff --repo "$REPO"`.
Pass the explicit PR identifier to both commands; use that PR's scope, not an unrelated checkout.

For a new PR, confirm the base branch and its remote, fetch it, and review the remote-based diff:

```bash
REPO='<target-owner>/<repository>'
BASE='<confirmed-base-branch>'
BASE_REMOTE='<remote-for-target-repository>'
gh repo view "$REPO" --json nameWithOwner,defaultBranchRef
git fetch "$BASE_REMOTE"
BASE_REF="$BASE_REMOTE/$BASE"
git log "$BASE_REF"..HEAD --oneline
git diff "$BASE_REF"...HEAD
```

Use the requested base or the target repository's default. If the remote-tracking base is
missing, fetch that branch explicitly before continuing. Check whether the intended head
already has a PR in the target repository; confirm the update scope if one exists.

### Step 2: Identify the Primary Change

Read the commits and diff. Identify the problem, affected behavior or consumers, motivation,
and any compatibility or operational consequences. Consult linked issues when available;
record alternatives only when they were actually considered.

State the primary change in one sentence before drafting. For an existing PR, distinguish its
original purpose from follow-up fixes and supporting dependency or tooling changes. Revise that
purpose only when the full diff and agreed scope support it.

Choose the title from that whole-PR purpose, not by copying the latest commit subject or leading
with a supporting detail. Finalize it after writing the description so both tell the same story.
For title-only requests, skip Step 3 and check the title against the existing description.
Use the repository's scopes and release policy:

| Type | Use |
| --- | --- |
| `fix` | Correct behavior or ship a dependency fix requiring a patch release |
| `feat` | Introduce a feature requiring a minor release |
| `refactor` | Restructure implementation without changing behavior |
| `chore` | Maintain tooling or configuration without requiring a package release |

Follow repository rules for other types and breaking changes. Dependency changes do not
automatically imply `chore`; check their release impact, especially for squash-merge titles.
Assess compatibility against the released consumer contract, not intermediate work on an
unreleased feature branch. Retain consequential migration constraints even without a major bump.

```text
fix(worker): update dependencies to address security advisories
```

### Step 3: Write the Description

Open with the problem and intended effect, using established motivation rather than inventing
a justification. Ask when missing facts prevent an accurate description. Write for a reviewer
who has not seen the conversation: use plain words, direct verbs, and consistent terms. Explain
necessary technical terms on first use; keep exact identifiers when they help locate the change.

Choose structure by review complexity:

- Use short prose for one straightforward behavior change whose motivation and impact fit in a
  few sentences.
- Use lightweight descriptive headings (such as `Why` and `What changed`) with grouped bullets
  for multiple distinct changes or review concerns that must be understood together. Group by
  concern, not by changed file.
- Surface material changes to guarantees, enforcement, or security exposure in the opening.
  Explain other consequential migration, compatibility, and rollout or rollback constraints in
  dedicated sections when they need more than a sentence.

Use file count as a cue to inspect breadth, not a threshold: many mechanically repeated files
can still be simple, while two files can carry substantial risk.

Keep details that help reviewers assess behavior, compatibility, or risk. Leave supporting
implementation detail to linked documentation or the diff. Leave routine file inventories,
commands, and work logs in the agent's response. Cut repetition, filler, and assurances already
obvious from the diff without dropping consequential constraints.

Add detail only when the change needs it, without a fixed template:

| Detail | Include when |
| --- | --- |
| Issue links or alternatives | They explain the motivation or decision |
| Evidence | Observed behavior, measurements, or screenshots support a review decision |
| Compact visual | It explains behavior or relationships more clearly than prose |
| Rollout or rollback limits | Compatibility, data changes, or consumer effects need explanation |

Scope measurements and comparisons to what was observed; redact sensitive screenshot content.
Distinguish intended behavior from verified outcomes, and reverting code from reversing data
changes or effects already observed by consumers. Describe known behavioral limitations as part
of the change. Omit empty sections and generic risk labels.

Keep test results, counts, CI status, and execution blockers in the agent's response or GitHub
Checks, unless the user or repository explicitly requires them in the description. In that
report, distinguish local results from CI and completed checks from pending, failed, or blocked
checks. Never invent passing results or treat an undiagnosed failure as unrelated.

Read [Title and Description Examples](references/description-examples.md) when choosing structure,
prioritizing a title, or simplifying dense wording. The examples illustrate style, not evidence
to copy into another PR.

### Step 4: Review the Requested Text

Before delivering, creating, or editing a PR, reread the requested text as a reviewer without
conversation history. Revise it until each applicable condition holds:

- The title names the primary change across the whole PR and follows its release policy.
- The opening explains the problem and intended effect; any headings expose review concerns.
- Sentences use concrete terms and direct verbs rather than unexplained internal terminology.
- Claims match the evidence, and compatibility, security, and rollout constraints remain intact.
- Required template content remains; work logs and execution status stay out unless required.
- Only the requested fields change; preserve an existing title or body outside the agreed scope.

Apply `clear-technical-writing` when available for sentence-level editing; these checks also
work without it. This is an editing pass, not an extra user-approval step or a PR-body checklist.
For writing-only requests, deliver only the requested title, description, or both, and stop here.

### Step 5: Publish or Update the PR

For code publication, run the checks required by the repository or requested by the user before
publishing by default. For a requested draft before validation, publish first, then run those
checks and report results to the user. Follow repository requirements when they mandate
validation first. Keep the PR a draft until a state change is authorized. Metadata-only edits
do not require new commits, a push, or unrelated validation.

Before publishing code, ensure only the intended changes are committed. Use `conventional-commit`
with the required authorization when available, or follow repository commit conventions directly.
Recheck that the committed diff matches the reviewed scope. For an existing PR, verify the local
branch and push destination match its head. Confirm the remote and push with required approval:

```bash
PUSH_REMOTE='<remote-for-head-repository>'
HEAD_BRANCH='<confirmed-head-branch>'
EXPECTED_HEAD=$(git rev-parse HEAD)
git push --set-upstream "$PUSH_REMOTE" "HEAD:refs/heads/$HEAD_BRANCH"
git ls-remote --exit-code "$PUSH_REMOTE" "refs/heads/$HEAD_BRANCH"
```

Confirm the reported remote commit matches `EXPECTED_HEAD`; stop on a mismatch or push failure.
For a fork PR, push to the fork and set `PR_HEAD` to `fork-owner:branch`; otherwise use the head
branch name.

Write the requested title and/or description to separate UTF-8 files in an approved temporary
location outside tracked project files. New PRs need both. Use a file-writing tool or a quoted
heredoc (`<<'EOF'`) so backticks and `$()` stay literal. Read the title as a quoted argument;
pass the description with `--body-file`. Include required template content yourself rather than
relying on gh to apply a template when a body file is supplied.

```bash
TITLE_FILE='<absolute-path-to-title.txt>'
BODY_FILE='<absolute-path-to-description.md>'
gh pr create --repo "$REPO" --base "$BASE" --head "$PR_HEAD" --draft \
  --title "$(cat "$TITLE_FILE")" --body-file "$BODY_FILE"
```

Omit `--draft` only when a ready PR is requested or required by repository policy.

For an existing PR, start from its fetched body and edit only the agreed content. Re-fetch
immediately before submitting and merge any intervening human edits instead of overwriting them.
Use the confirmed numeric `PR_NUMBER` and target repository:

```bash
gh pr edit "$PR_NUMBER" --repo "$REPO" --body-file "$BODY_FILE"
```

Change the title only when requested or required by the agreed scope. If an older GitHub CLI
fails with the Projects classic deprecation error, prefer upgrading it. Otherwise use the
REST API with file-based input rather than interpolating the Markdown into a shell string:

```bash
gh api --method PATCH "repos/$REPO/pulls/$PR_NUMBER" \
  --field body=@"$BODY_FILE"
```

For an authorized title edit, also pass `--title "$(cat "$TITLE_FILE")"` to `gh pr edit`, or
`--raw-field title="$(cat "$TITLE_FILE")"` to the REST command.
For a title-only edit, omit the body argument. Otherwise leave the title unchanged.

After creation, use the returned PR number or URL to read it back. After editing, read back
the same PR. Confirm the title, body, base, head, `headRefOid`, and draft state in the target
repository. For code publication, compare `headRefOid` to `EXPECTED_HEAD`; for metadata-only
edits, confirm the head and draft state stayed unchanged. Report the URL and validation status
in the agent's response, separately from the PR description.
