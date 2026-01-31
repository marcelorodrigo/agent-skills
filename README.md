# Agent Skills

A collection of skills for AI coding agents. Skills are packaged instructions and scripts that extend agent capabilities.

Skills available in this repository follow the [Agent Skills format](https://agentskills.io/).

## Available Skills

| Skill               | Description                                                                 |
|---------------------|-----------------------------------------------------------------------------|
| `conventional-commit` | Create commit messages following [Conventional Commits](https://www.conventionalcommits.org/) |
| `create-pr`         | Create pull requests following best engineering practices                   |

### conventional-commit

**Killer features:** Analyzes your changes to understand what was modified, explains the **why** (not just what), contrasts with previous behavior, and crafts commits that tell a short story. Follows Conventional Commits with proper types, scopes, and formatting for readable git history and automated versioning.

```bash
npx skills add https://github.com/marcelorodrigo/agent-skills --skill conventional-commit
```

### create-pr

**Killer features:** Reviews your commits and diffs to deeply understand changes, explains the **motivation** behind them, considers alternative approaches, and structures PRs for faster, better reviews. Creates clear descriptions focused on context reviewers need (not redundant test plans or checklists). Uses draft PRs for early feedback.

```bash
npx skills add https://github.com/marcelorodrigo/agent-skills --skill create-pr
```

## Installation

Install skills from this repository using the [add-skill](https://github.com/vercel-labs/add-skill) CLI:

```bash
# Install all skills
npx add-skill marcelorodrigo/agent-skills

# Install a specific skill
npx add-skill marcelorodrigo/agent-skills --skill conventional-commit
npx add-skill marcelorodrigo/agent-skills --skill create-pr

# List available skills
npx add-skill marcelorodrigo/agent-skills --list
```

## Repository Guide

See [AGENTS.md](./AGENTS.md) for repository structure, code style guidelines, and conventions.
