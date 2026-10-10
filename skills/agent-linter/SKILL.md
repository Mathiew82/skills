---
name: agent-linter
description: "Audits all the context an AI agent receives (AGENTS.md, CLAUDE.md, skills, referenced docs, MCP configs) and generates an executive report of contradictions, ambiguities, redundancies, and structural issues. Ranks findings by impact, calculates a context health score, and offers a detailed report on demand. It does not apply changes on its own: at the end it asks the user whether to apply them. The report is always generated in the user's language."
---

# Agent Linter

## What this skill does

Analyzes **all the context an AI agent reads before acting** and generates an **executive, direct, and actionable** report with:

- Context health score (0-10)
- Main diagnosis (root cause of the problems)
- Priority actions (in order)
- Proposed canonical structure (when applicable)
- Top 3 by token impact
- Top behavioral-risk findings
- Compact table with the rest of the problems
- Global metrics (tokens, density, potential savings)

**Report philosophy**: this is an **executive report**, not a forensic one. The user wants to know what to fix first, not read a treatise. It should be **1/3** the length of a detailed report.

**Language rule**: detect the language the user is speaking and generate the entire report in that language. If the user speaks Spanish, output the report in Spanish. If they speak English, in English. And so on for any language.

**Important**: this skill **does not modify any file**. It only generates a report. At the end it offers options, including generating a detailed report if the user asks for it.

---

## Phase 1 — File discovery

Locate and read, in this order, all the files the agent receives as context:

### 1.1 Known root files
- `AGENTS.md`
- `CLAUDE.md`
- `.cursorrules`
- `.cursor/rules/*.mdc`
- `.github/copilot-instructions.md`
- `README.md` (only if the agent reads it — check references)

### 1.2 Referenced files (transitive, up to depth 2)
Look in the previous files for any reference like:
- "Read X"
- "See X"
- "Check X"
- "Also see X"
- `[text](path/file.md)`
- `@path/file.md`

Add those files to the list and repeat the process one more time (depth 2 maximum, to avoid loops).

### 1.3 Installed skills
Locate the agent's skills (depending on the framework):
- Claude: `.claude/skills/*/SKILL.md`
- Cursor: `.cursor/rules/*.mdc`
- OpenAI GPTs: `skills/` folder
- Generic: any `SKILL.md` in the project

Extract from each skill: `name`, `description`, internal instructions, exposed tools.

### 1.4 MCP configuration files
- `mcp.json`
- `claude_desktop_config.json`
- `.mcp/config.json`
- Any config that defines MCP servers

### 1.5 Files explicitly given by the user
If the user invokes the skill with `--include path/file.md`, add it to the list.

### 1.6 Always exclude
- `node_modules/`, `.git/`, `dist/`, `build/`, `vendor/`, `.next/`, `venv/`
- Binary files
- Files > 500KB (warn the user if they are skipped)

**Phase output**: list of files with their path, approximate token size, and type (root / referenced / skill / config).

---

## Phase 2 — Atomic rule extraction

You (the agent) extract the atomic rules from each file yourself. There is no external script. Read them fully and apply this criterion:

An atomic rule is an imperative instruction, verifiable, with a single subject of action.

Examples:
- ✅ "Use single quotes in JavaScript." → atomic rule
- ❌ "Write clean, well-structured code with good practices, using strict TypeScript and preferring pure functions." → 4 mixed atomic rules

Format of each rule:
```
{
  "id": "rule_001",
  "text": "Use single quotes in JavaScript.",
  "file": "CONVENTIONS.md",
  "line": 14,
  "type": "style | workflow | security | format | tone | structure | other"
}
```

Rules for this phase:
- Do NOT extract titles, code blocks, or paragraphs without an imperative verb.
- Do NOT extract filler, disclaimers, motivations, or acknowledgments.
- If a rule is literally repeated across files, create one per file (same text, different id). Redundancy detection happens in Phase 3.
- If you are unsure whether something is a rule or not, treat it as a rule. Better a false positive here than a false negative.

**Phase output**: full list of atomic rules with their metadata.

---

## Phase 3 — Problem analysis

For each type of problem, apply the described logic. **Do not use hardcoded dictionaries**. Use semantic reasoning.

### 3.1 Contradictions and ambiguities

**Two-step strategy for efficiency:**

**Step 1 — Pre-filter:**
- Compare pairs of atomic rules **that come from different files**.
- Discard pairs that clearly have no relation (different type AND unrelated topics).
- Only pairs that could conflict move on to step 2.

**Step 2 — Fine analysis:**
For each filtered pair, use the `prompts/analyze-pair.md` prompt and get a structured verdict.

### 3.2 Redundancy detection

**Grouping strategy:**

1. Take all atomic rules and group them by semantic similarity.
2. Each group with more than 1 rule is a **candidate redundancy group**.
3. Inside each group, analyze with the `prompts/cluster-rules.md` prompt.
4. Designate a **canonical rule** (the clearest, most complete, most specific one).
5. Mark the others as candidates for removal/merging.

**Mandatory quantification**: for each redundancy group, calculate:
- Total tokens of the group.
- Tokens of the canonical rule.
- Tokens that would be saved by removing the redundant ones.

### 3.3 Workflow incompatibilities

Look for rules that define mutually exclusive workflows:
- "Before X, ask the user" vs "Act autonomously without asking for confirmation"
- "Run commands without checking" vs "Always check before running"
- "Respond directly" vs "Ask for confirmation before responding"

### 3.4 Broken references

For each reference detected in Phase 1.2, verify the target file exists. If not, report it. Mark as **essential** if the reference is to something necessary for the skill to work, or **cosmetic** if it is an informational link.

### 3.5 Outdated rules

Cross-check rules against the real project state:
- Does the rule mention a technology not in `package.json` / `requirements.txt` / etc.?
- Does it mention a different version than the installed one?
- Does it mention a file/folder that no longer exists?

### 3.6 Context overload

Calculate:
- Total tokens of the entire agent context.
- Tokens per file.
- Alert threshold: 30,000 tokens (configurable).

### 3.7 Scope conflicts

Detect when a specific file overrides a rule from a root file without saying so explicitly.

### 3.8 Orphan or malformed rules

- Rules that are not verifiable ("be a good programmer").
- Rules without a clear subject ("improve readability").

### 3.9 Impact weight calculation

Before generating the report, calculate the impact weight of each finding. Impact is calculated like this:

- **Conflicts**: weight = severity (3=critical, 2=high, 1=medium, 0.5=low) × number of affected rules × estimated frequency (3=high, 2=medium, 1=low).
- **Redundancies**: weight = savings in tokens + (number of involved rules × 10).
- **Broken references**: weight = 100 if essential, 20 if cosmetic.
- **Context overload**: weight = tokens over the threshold / 100.

The 3 findings with the highest weight are the **Top 3 by token impact**.

### 3.10 Health score calculation

Start at 10 points. Subtract:

- Critical conflicts: -3 each (max -6)
- High conflicts: -1.5 each (max -3)
- Medium conflicts: -0.5 each (max -1.5)
- Ambiguities: -0.3 each (max -1)
- Redundancies: -0.2 per group (max -1)
- Broken references: -0.5 essential, -0.1 cosmetic (max -1.5)
- Overload: -1 if >30K tokens, -2 if >50K

Floor: 0. Round to one decimal.

**Important context**: the score should NOT be read as "number of problems", but as "operational health of the context". A context with 10 minor redundancies but no critical conflict should score ≥7. A context with 1 critical conflict and nothing else should score ≤6.

Interpretation:
- 9-10: excellent.
- 7-8.9: good, room for improvement.
- 5-6.9: fair, several problems to solve.
- 3-4.9: problematic, requires attention soon.
- 0-2.9: critical, the context is sabotaging the agent.

### 3.11 Root-cause detection and priority actions

Before generating the report, identify:

1. **The main root cause** of most conflicts. It is usually structural: lack of hierarchy between skills, absence of a priorities file, domain overlap, etc.
2. **The 2-3 priority actions** that, applied in order, would solve most problems. Each action must have a measurable impact ("solves 5 conflicts", "saves 1,000 tokens", etc.).
3. **The estimated impact on the health score** if those actions are applied.

This is reported in the "Main diagnosis" section of the report.

---

## Phase 4 — Report generation

Use the `prompts/build-report.md` prompt to generate the final report.

**Philosophy**: **executive** report, not forensic. Direct, concise, actionable. It should be **1/3** the length of a detailed report.

**Language rule**: generate the entire report in the same language the user is speaking. If the user writes in Spanish, the report is in Spanish. If they write in English, in English. Apply the same for any other language.

**Mandatory structure of the executive report**:

1. Health score (X/10) with impact table by category.
2. **Main diagnosis** + priority actions (with estimated impact on the score).
3. **Proposed canonical structure** (when applicable).
4. Top 3 by token impact.
5. Top behavioral-risk findings (high conflicts that do not make the Top 3 by tokens).
6. Compact table with the rest of the problems.
7. Global metrics and heaviest files.
8. Closing question with options (including "detail").

**Detail option**: at the end of the report, the user can ask for **"detail"** or **"detailed report"**. In that case, generate the full report with:

- Each conflict expanded with extensive explanation.
- Each redundancy group with all rules and justification.
- Atomic rules listed per file.
- Justification for each recommendation.
- Concrete examples when they add value.

**Execution modes**:
- **`--summary`** (default): executive report.
- **`--detail`**: full report directly.

If the user does not specify, assume `--summary`.

**Report style rules**:
- No long paragraphs. Use lists and tables.
- Each section should fit on screen without excessive scrolling.
- The "Top 3 findings" are the most important. Take care of them.
- In the problems table, be telegraphic.
- Do not repeat information across sections.
- If a datum is uncertain, write "≈" or "review manually".
- Never invent. If there are no problems in a category, write "None detected."

**Mandatory report closing** (translated to the user's language):

```markdown
---

## ⚠️ This skill has NOT modified any file.

What do you want to do?
- **"apply all"** → I execute all actions.
- **"apply only [section]"** → e.g., "apply only high conflicts".
- **"apply one by one"** → I review each change with you.
- **"no"** → leave everything as is.
- **"detail"** or **"detailed report"** → I generate the full report with all explanations, examples, and justifications.
```
