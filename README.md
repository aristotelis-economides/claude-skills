# Claude Skills

A collection of skills for extending Claude's capabilities in agentic workflows.

## What are Skills?

Skills are modular packages that provide Claude with specialized knowledge, workflows, and tools for specific domains. They transform Claude from a general-purpose assistant into a specialized agent equipped with procedural knowledge.

## Available Skills

| Skill | Description |
|-------|-------------|
| [agentic-harness](./agentic-harness/) | Design principles for building effective agentic systems and agent-native applications. Use when initializing CLAUDE.md files, designing agent architectures, or setting up long-running autonomous coding projects. |

## Using Skills

### With Claude Code

Place skill directories in your project or reference them in your CLAUDE.md:

```markdown
## Skills
Read `/path/to/skills/agentic-harness/SKILL.md` when setting up new projects.
```

### With Claude.ai

Skills can be uploaded as .skill files (zip archives) to Claude.ai's skills feature.

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
