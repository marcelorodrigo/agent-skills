# PR Title and Description Examples

Use these examples to choose a presentation, not as mandatory templates. Replace illustrative
facts and measurements with observed facts. Follow required repository sections even when an
example uses plain prose. Keep execution status in the agent's response or GitHub Checks.
The conventional titles below assume the stated change type fits the repository's release policy.

## Small Configuration Fix

Use a short explanation when motivation and impact fit in a few sentences. Skip a diagram
when it would repeat the configuration diff.

Title: `chore(lint): check unused imports in editor lint runs`

```markdown
Enable the existing unused-imports rule in editor lint runs as well as CI. Developers currently
see those errors only after opening a PR. The rule and its severity stay the same.
```

## Dependency Security Fix

Scope security improvements without implying every advisory is fixed. Choose the title using
the repository's release policy; use `fix` when this change requires a patch release.

Title: `fix(worker): update dependencies to address security advisories`

```markdown
Update compatible dependencies to reduce audit findings from 18 to 11, including high-severity
findings from 6 to 3. These counts cover the local dependency tree, not open GitHub alerts.

The remaining high findings come through an exact-pinned build-tool family that needs a
coordinated upgrade. This change leaves public peer ranges unchanged.
```

## Multiple Review Concerns

Use headings and grouped bullets when reviewers must understand distinct concerns together.
Separate the API, processing, and user-facing behavior rather than listing each changed file.

Title: `feat(inventory): add CSV stock imports`

```markdown
### Why

Store managers currently enter stock adjustments one item at a time. Let them upload a CSV
file and review rejected rows before the import changes stock levels.

### What changed

- **Upload API:** Accept stock-adjustment files and return an import identifier.
- **Processing:** Validate rows and store rejected entries without changing their stock levels.
- **Review screen:** Show accepted and rejected rows before the manager confirms the import.

### Rollout

Deploy the import worker before enabling CSV uploads. Keep manual adjustments available while
the import flow is disabled.
```

## Primary Change Versus Supporting Detail

Suppose the PR adds a bulk order-submission endpoint. Request normalization and retry helpers
support that behavior but do not describe the capability consumers receive.

| Title | Assessment |
| --- | --- |
| `feat(api): add request normalization and retry helpers` | Buries the primary change |
| `feat(api): add bulk order submission` | Names the consumer-facing capability |

For an existing PR that adds CSV downloads, accompanying test fixes and dependency updates do
not replace that feature as its purpose. Use a title such as `feat(export): add CSV downloads`
when supported by the complete diff and release policy, rather than copying the last fix commit.

## Dense Wording Versus Plain Language

Suppose the diff lets workspace administrators invite members to their own workspace, a task
that previously required an organization administrator. Organization-wide access stays restricted.

Dense: "Decouple membership provisioning from organization-scoped administration through
role-stratified authorization predicates and tenant-constrained subject resolution."

Title: `feat(workspaces): let workspace administrators invite members`

```markdown
### Why

Workspace administrators must ask an organization administrator to invite each new member.

### What changed

- Let workspace administrators invite members to their own workspace.
- Keep organization-wide access restricted to organization administrators.

Existing organization administrators keep their current permissions.
```

Keep meaningful scope limits such as "their own workspace" while simplifying the wording.
Explain the permission boundary instead of listing authorization implementation details.

## Behavior Ordering

Use a compact visual when ordering or ownership is clearer as a diagram.

Title: `fix(checkout): reserve inventory before confirming orders`

````markdown
Reserve inventory before confirming an order. Confirmation currently happens first, so an
inventory shortage can reject an order after the customer receives a confirmation.

```text
submit order
  validate order
  reserve inventory
  confirm order
```
````

## Risky Migration

Describe rollout boundaries and the point beyond which reverting code is insufficient.

Title: `feat(billing): add normalized invoice-reference writes`

```markdown
Add a nullable normalized invoice-reference column and dual writes as the first step toward
preventing duplicate matches caused by casing and whitespace. Legacy reads stay in place;
switching reads and removing the old column remain separate changes.

### Rollout and rollback

Deploy the schema before enabling dual writes. Keep legacy reads until the backfill and
consistency checks complete. Reverting the application leaves the new column in place;
dropping it requires a separate data-preservation decision.
```
