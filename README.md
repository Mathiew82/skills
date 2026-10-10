# Skills

A collection of skills for AI agents. Each skill is a self-contained set of instructions that extends an agent's capabilities without external dependencies.

## Available skills

| Skill | Description | Install |
|---|---|---|
| [agent-linter](./skills/agent-linter) | Audits all the context an AI agent receives (AGENTS.md, CLAUDE.md, skills, referenced docs, MCP configs) and generates an executive report of contradictions, ambiguities, redundancies, and structural issues. | npx skills add Mathiew82/skills --skill agent-linter |

## Installation

### Install a specific skill

    npx skills add Mathiew82/skills --skill <skill-name>

Example:

    npx skills add Mathiew82/skills --skill agent-linter

### Install all skills from this repository

    npx skills add Mathiew82/skills --all

## Usage

Once installed, invoke the skill from your agent (Claude Code, Cursor, etc.) by its name.

Example:

    run agent-linter on this project

The skill will analyze your agent's context and generate a report. It will not modify any file unless you explicitly ask it to.

## Repository structure

    skills/
    ├── README.md
    ├── LICENSE
    ├── .gitignore
    └── skills/
        └── agent-linter/
            ├── SKILL.md
            └── prompts/
                ├── analyze-pair.md
                ├── cluster-rules.md
                └── build-report.md

Each skill lives in its own folder under `skills/`. A skill is defined by its `SKILL.md` file, which contains the frontmatter (name, description, license) and the instructions the agent follows.

## Contributing

This is a personal collection. If you find a bug or have a suggestion, feel free to open an issue.

## License

MIT — see [LICENSE](./LICENSE) for details.
