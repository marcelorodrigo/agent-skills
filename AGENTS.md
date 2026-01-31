# AGENTS.md - Agent Skills Repository

This repository contains AI agent skills following the [Agent Skills format](https://agentskills.io/).

## Repository Structure

```
skills/
  <skill-name>/
    SKILL.md      # Skill definition with frontmatter
```

## Commands

This is a documentation/skill repository with no build, lint, or test commands.

## Code Style Guidelines

### SKILL.md Files

- **Frontmatter**: Required YAML frontmatter with `name` and `description` fields
- **Format**: Use markdown with clear sections and code blocks
- **Line Length**: Keep lines under 100 characters

### File Naming

- Skill directories: kebab-case (e.g., `create-pr`, `commit`)
- Skill files: `SKILL.md` (uppercase, consistent across all skills)

### Writing Style

- Use imperative mood for instructions
- Include code examples in fenced blocks with language tags
- Organize with clear headings (##, ###)
- Explain **what** and **why**, not just how

### Git Conventions

All commits follow [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <subject>
```

Types: `feat`, `fix`, `refactor`, `docs`, `style`, `test`, `chore`, `ci`, `build`, `deps`, `perf`, `revert`

Branch naming: `<type>/<short-description>`

Examples:
- `feat/add-todo-skill`
- `fix/commit-skill-examples`

### PR Guidelines

- One PR per skill or fix
- Use draft PRs for early feedback
- Title follows commit convention format
- Description explains motivation and context

## Adding New Skills

1. Create directory: `skills/<skill-name>/`
2. Add SKILL.md with frontmatter and documentation
3. Follow existing skills as templates
4. Commit with appropriate type and scope
