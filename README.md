# Claude Skills

A collection of skills I use with [Claude](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview).

#TODO: generalize for all agents.

## Available Skills

| Skill | Description |
|-------|-------------|
| [agentic-harness](./agentic-harness/) | Design principles for building effective agentic systems and agent-native applications. Use when initializing CLAUDE.md files, designing agent architectures, or setting up long-running autonomous coding projects. |

## Installation

### Claude Code

Add this repo as a skill source in your `~/.claude/settings.json`:

```json
{
  "skills": [
    "/path/to/agent-skills"
  ]
}
```

Or reference specific skills in your project's `CLAUDE.md`:

```markdown
## Skills
Read `/path/to/agent-skills/agentic-harness/SKILL.md` when setting up new agentic projects.
```

### Claude.ai

Skills can be uploaded as .skill files (zip archives) to Claude.ai's skills feature.

## Example: Starting a New Agentic Project

With Claude Code, start a new project and trigger the skill:

```bash
# Create project directory
mkdir my-agentic-app && cd my-agentic-app

# Start Claude Code
claude

# Then tell Claude:
# "Set up this project for autonomous multi-session development.
#  Use the agentic-harness skill to create the scaffolding."
```

Claude will read the skill and create:
- `features.json` with testable feature specifications
- `sessions.md` for session logging
- `progress.md` for project state tracking
- `init.sh` for environment setup
- Initial git commit

Subsequent sessions start with:
```
"Continue working on this project. Read sessions.md and progress.md to orient, then pick up where the last session left off."
```

## Skill Structure

Each skill follows this structure:

```
skill-name/
├── SKILL.md              # Main instructions (required)
└── references/           # Supporting documentation (optional)
    ├── templates.md
    └── domain-specific.md
```

## Contributing

To add a new skill:

1. Create a directory with your skill name
2. Add a SKILL.md with YAML frontmatter containing `name` and `description`
3. Include any supporting references or assets
4. Update this README

## License

MIT
