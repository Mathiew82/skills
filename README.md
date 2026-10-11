<div align="center">
    <img src="./logo.png" width="320" alt="skills"><br>
    <h1>AI Agent Skills</h1>
    <p>A collection of self-contained skills that extend AI agents.</p>
</div>

## Available skills

| Skill | Description | Install |
|---|---|---|
| [agent-linter](./skills/agent-linter) | Audits all the context an AI agent receives (AGENTS.md, CLAUDE.md, skills, referenced docs, MCP configs) and generates an executive report of contradictions, ambiguities, redundancies, and structural issues. | npx skills add https://github.com/mathiew82/skills --skill agent-linter |

## Installation

### Install a specific skill

    npx skills add https://github.com/mathiew82/skills --skill <skill-name>

Example:

    npx skills add https://github.com/mathiew82/skills --skill agent-linter

### Install all skills from this repository

    npx skills add https://github.com/mathiew82/skills --all

## Usage

Once installed, invoke the skill from your agent (Claude Code, Cursor, etc.) by its name.

Example:

    run agent-linter on this project

The skill will analyze your agent's context and generate a report. It will not modify any file unless you explicitly ask it to.

### Report modes

agent-linter generates an executive report by default. You can ask for other formats in plain language — no need to memorize commands.

- **Detailed report**: ask for "detail" or "a detailed report". You'll get the full version with the original passages side by side, so you can decide which rule to keep.
  Example: `run agent-linter and give me the detailed report`
- **Plain language report**: ask for "plain language", "no jargon", or "explain like I'm not a developer". You'll get the same executive report but without technical terms.
  Example: `run agent-linter but explain it in plain language`
- **Apply changes**: after the report, you can reply with "apply all", "apply only [section]", or "apply one by one". The skill will never modify anything unless you ask it to.

You don't need to know any specific commands. Just ask for what you want in your own words. The skill detects the intent, not the exact phrasing.

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
