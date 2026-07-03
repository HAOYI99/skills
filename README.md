# skills

Agent skills for [skills.sh](https://skills.sh) — works with any AI coding agent that supports the skills.sh standard (GitHub Copilot, Cursor, Windsurf, and more).

[![skills.sh](https://skills.sh/b/HAOYI99/skills)](https://skills.sh/HAOYI99/skills)

## Install a skill

```sh
skills install HAOYI99/skills/<skill-name>
```

## Available skills

### Work

| Skill | Description |
|---|---|
| [notion-mcp](./notion-mcp/SKILL.md) | Set up and operate Notion via MCP — init config, search, create/update pages and tasks, sync, comment. |

## Usage

Each skill lives in its own directory with a `SKILL.md`. The agent reads it at runtime to know what to do.

## Contributing

Open an issue or PR. New skills go in a dedicated subdirectory with a `SKILL.md`.
