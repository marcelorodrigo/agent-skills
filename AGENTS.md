# AGENTS.md - Agent Skills Repository

This repository contains AI agent skills following the [Agent Skills format](https://agentskills.io/).

## Repository Structure

```
skills/
  <skill-name>/
    SKILL.md      # Skill definition with YAML frontmatter and documentation
```

## Commands

This is a documentation/skill repository with no build, lint, or test commands.

### Validation

Skills are validated using the `skills-ref` validator from the [agentskills](https://github.com/agentskills/agentskills) project:

```bash
# Install the validator
pip install git+https://github.com/agentskills/agentskills.git#subdirectory=skills-ref

# Validate a single skill
skills-ref validate skills/<skill-name>

# Validate all skills
for skill_dir in skills/*/; do
  skills-ref validate "$skill_dir"
done
```

Validation runs automatically on pull requests via GitHub Actions (`.github/workflows/validate-skills.yml`).

## Code Style Guidelines

### SKILL.md Files

**Frontmatter** (required at the top):
```yaml
---
name: skill-name
description: Brief description of what the skill does
license: MIT
compatibility: Optional requirements (e.g., "Requires GitHub CLI")
metadata:
  version: "1.0.0"
---
```

- **name**: kebab-case, matches directory name
- **description**: One sentence, imperative mood, explains when to use the skill
- **license**: MIT (standard for this repository)
- **compatibility**: Optional, list external requirements
- **metadata.version**: Semantic version

**Content Guidelines**:
- Use clear headings (`#`, `##`, `###`)
- Include code examples in fenced blocks with language tags
- Keep all lines under 100 characters
- Use tables for structured data (types, options, examples)

### File Naming

- **Skill directories**: kebab-case (e.g., `conventional-commit`, `create-pr`)
- **Skill files**: `SKILL.md` (uppercase, consistent across all skills)
- **Workflow files**: kebab-case with `.yml` extension

### Writing Style

- Use **imperative mood** for instructions ("Run tests", not "You should run tests")
- Include **code examples** in fenced blocks with language specification
- Explain **what** and **why**, not just how
- Use **present tense** throughout

### Git Conventions

All commits follow [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <subject>
```

**Types** (in order of frequency):
- `feat`: New skill or major feature
- `fix`: Bug fix or correction
- `docs`: Documentation updates
- `style`: Formatting, whitespace, markdown alignment
- `refactor`: Code restructure without behavior change
- `ci`: CI/CD configuration changes
- `deps`: Dependency updates
- `test`: Test additions or updates
- `chore`: Maintenance tasks
- `perf`: Performance improvements
- `build`: Build system changes
- `revert`: Revert previous commit

**Scope**: Use skill name for skill changes (e.g., `feat(conventional-commit)`), or area for repo changes (e.g., `ci(deps)`)

**Subject rules**:
- Maximum 70 characters
- Use imperative, present tense
- Capitalize first letter
- No period at the end

**Branch naming**: `<type>/<short-description>`
- Examples: `feat/add-todo-skill`, `fix/commit-examples`, `docs/readme-update`

### PR Guidelines

- **One PR per skill or logical change**
- Use **draft PRs** for early feedback
- **Title** follows commit convention format
- **Description** explains motivation, context, and any alternatives considered
- Reference related issues if applicable

## Adding New Skills

1. Create directory: `mkdir skills/<skill-name>`
2. Add `SKILL.md` with required frontmatter and documentation
3. Follow existing skills as templates (see `skills/conventional-commit/` or `skills/create-pr/`)
4. Validate locally: `skills-ref validate skills/<skill-name>`
5. Commit with appropriate type and scope
6. Create PR following PR guidelines

## References

- [Agent Skills Format](https://agentskills.io/)
- [Conventional Commits Specification](https://www.conventionalcommits.org/)
- [GitHub Actions Workflow](.github/workflows/validate-skills.yml)
