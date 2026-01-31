# Agent Skills

A collection of skills for AI coding agents. Skills are packaged instructions and scripts that extend agent capabilities.

Skills available in this repository follow the [Agent Skills format](https://agentskills.io/).

## Available Skills

| Skill               | Description                                                                 |
|---------------------|-----------------------------------------------------------------------------|
| `conventional-commit` | Create commit messages following [Conventional Commits](https://www.conventionalcommits.org/) |
| `create-pr`         | Create pull requests following best engineering practices                   |
| `filament-pro`      | Build Laravel admin panels with Filament v5 using server-driven UI          |
| `spring-boot-testing` | Expert Spring Boot 4 testing with best practices and modern Java 25 features |

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

### filament-pro

**Killer features:** Comprehensive guide for building Laravel admin panels with Filament v5. Covers server-driven UI, schema-based forms, tables with filters and actions, relation managers, testing with Pest, multi-tenancy, and authorization. Includes 10 detailed reference files and complete code examples for rapid development.

```bash
npx skills add https://github.com/marcelorodrigo/agent-skills --skill filament-pro
```

### spring-boot-testing

**Killer features:** Expert guide for testing Spring Boot 4 applications with Java 25 features. Selects optimal test slices (@WebMvcTest, @DataJpaTest, @SpringBootTest), uses MockMvcTester with AssertJ-style assertions, integrates Testcontainers for real database testing, applies production-scenario testing, and maintains 80% coverage. Includes complexity assessment, helper methods, and @DisplayName best practices with focused reference files.

```bash
npx skills add https://github.com/marcelorodrigo/agent-skills --skill spring-boot-testing
```

## Installation

Install skills from this repository using the [skills](https://github.com/vercel-labs/skills) CLI:

```bash
# Install all skills
npx skills add marcelorodrigo/agent-skills

# Install a specific skill
npx skills add marcelorodrigo/agent-skills --skill conventional-commit
npx skills add marcelorodrigo/agent-skills --skill create-pr
npx skills add marcelorodrigo/agent-skills --skill filament-pro
npx skills add marcelorodrigo/agent-skills --skill spring-boot-testing

# List available skills
npx skills add marcelorodrigo/agent-skills --list
```

## Repository Guide

See [AGENTS.md](./AGENTS.md) for repository structure, code style guidelines, and conventions.
