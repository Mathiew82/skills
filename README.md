# Skills

A collection of skills for AI agents. Each skill is a self-contained set of instructions that extends an agent's capabilities without external dependencies.

## Available skills

| Skill | Description | Install |
|---|---|---|
| [`agent-linter`](./skills/agent-linter) | Audits all the context an AI agent receives (AGENTS.md, CLAUDE.md, skills, referenced docs, MCP configs) and generates an executive report of contradictions, ambiguities, redundancies, and structural issues. | `npx skills add Mathiew82/skills --skill agent-linter` |

## Installation

### Install a specific skill

```bash
npx skills add Mathiew82/skills --skill <skill-name>
