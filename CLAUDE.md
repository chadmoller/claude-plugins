# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is a Claude Code plugins repository. Plugins extend Claude Code with custom commands, agents, skills, and hooks.

## Structure

Each plugin lives in its own subdirectory and must contain a `.claude-plugin/plugin.json` manifest:

```
<plugin-name>/
  .claude-plugin/
    plugin.json        # { "name": "...", "description": "..." }
  commands/            # Slash commands (.md files)
  agents/              # Subagent definitions (.md files)
  skills/              # Skill definitions (.md files)
  hooks/               # Hook scripts
```

## Plugin Development

Use the `plugin-dev:*` skills for guided workflows:
- `/plugin-dev:create-plugin` — end-to-end plugin creation
- `/plugin-dev:command-development` — slash commands
- `/plugin-dev:hook-development` — PreToolUse/PostToolUse/Stop hooks
- `/plugin-dev:agent-development` — subagent definitions
- `/plugin-dev:skill-development` — skills
- `/plugin-dev:plugin-structure` — manifest and layout conventions

## Commands

Command files are Markdown with YAML frontmatter:

```markdown
---
description: <shown in /help>
argument-hint: <arg placeholder>
allowed-tools: [Read, Write, Bash, ...]
---

# Command Title

Body explains what Claude should do when this command is invoked.
```

## Skills

Skills live in `<plugin>/skills/<skill-name>/SKILL.md` and use YAML frontmatter with `name` and `description` (third-person, with specific trigger phrases). The body uses imperative form. See `personal-tools/skills/plan-branch/SKILL.md` for an example: it enters plan mode, derives a git branch name from the plan title, and commits the plan into `plans/` on that branch.
