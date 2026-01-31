# Agent Skills

A collection of skills for AI coding agents. Skills are packaged instructions and scripts that extend agent capabilities.

Skills available in this repository follow the [Agent Skills format](https://agentskills.io/).

## Available Skills

| Skill       | Description                                                                 |
|-------------|-----------------------------------------------------------------------------|
| `conventiona;l-commit`    | Create commit messages following [Conventional Commits](https://www.conventionalcommits.org/) |
| `create-pr` | Create pull requests following best engineering practices                   |

## Installation

Install skills from this repository using the [add-skill](https://github.com/vercel-labs/add-skill) CLI:

```bash
# Install all skills
npx add-skill marcelorodrigo/agent-skills

# Install a specific skill
npx add-skill marcelorodrigo/agent-skills --skill commit

# List available skills
npx add-skill marcelorodrigo/agent-skills --list
```

## Repository Guide

See [AGENTS.md](./AGENTS.md) for repository structure, code style guidelines, and conventions.
