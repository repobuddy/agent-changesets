# agent-changesets

Skills for AI agents to work with [changesets](https://github.com/changesets/changesets) — the versioning and changelog tool for JavaScript/TypeScript packages.

## Skills

| Skill | Description |
| ----- | ----------- |
| **add-changeset** | Analyse the current change, determine affected packages and bump type, and write a changeset file directly. |
| **setup-changesets** | Set up changesets in a new or existing repository, including CI release workflow. |

## Installation

```bash
# Install all skills globally
npx skills add repobuddy/agent-changesets --all -g

# Install a specific skill
npx skills add repobuddy/agent-changesets --skill add-changeset -g

# Install for a specific agent
npx skills add repobuddy/agent-changesets --skill add-changeset -a claude-code -g
```
