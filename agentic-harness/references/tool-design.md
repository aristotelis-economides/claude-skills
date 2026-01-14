# Tool Design Patterns

Principles for designing tools in agent-native applications.

## Table of Contents

- [Atomic Over Bundled](#atomic-over-bundled)
- [Tool Design Principles](#tool-design-principles)
- [When to Add Domain Tools](#when-to-add-domain-tools)
- [Files as Universal Interface](#files-as-universal-interface)
- [Context File Pattern](#context-file-pattern)

---

## Atomic Over Bundled

**Less granular (bundles judgment into tool):**
```
classify_and_organize_files(files)
→ You wrote the decision logic
→ Agent executes your code
→ To change behavior, refactor code
```

**More granular (agent makes decisions):**
```
Tools: read_file, write_file, move_file, list_dir
Prompt: "Organize downloads by file type..."
→ Agent decides how to organize
→ To change behavior, edit prompt
```

The granular approach enables emergent capability—agents can combine primitives in ways you didn't anticipate.

---

## Tool Design Principles

1. **Atomic primitives first**: Start with bash, file operations, basic storage
2. **Domain tools for vocabulary**: `create_note` teaches what "note" means in your system
3. **Keep primitives available**: Domain tools are shortcuts, not gates
4. **CRUD completeness**: Every entity needs create, read, update, delete operations
5. **Agent-tool parity**: Whatever users can do through UI, agents should achieve through tools

### Audit for Parity

Review all UI actions and ensure corresponding agent tools exist:

| UI Action | Agent Tool Required |
|-----------|---------------------|
| Create project | `create_project` tool |
| Mark task complete | `update_task` tool |
| Upload file | `write_file` or `upload` tool |
| Send message | `send_message` tool |

If users can do it, agents must be able to achieve it.

---

## When to Add Domain Tools

Add domain-specific tools when:

1. **Establishing vocabulary** - What is a "note", "task", "project" in this system?
2. **Adding guardrails** - Destructive operations need confirmation or validation
3. **Optimizing hot paths** - Frequently-used operations benefit from speed/cost optimization
4. **Preventing common errors** - Validation catches mistakes before they propagate

### Domain Tool Design Rules

- Represent ONE conceptual user action
- Include mechanical validation (type checking, required fields)
- Keep judgment in prompts, not tool code
- Return structured results agents can reason about

**Good domain tool:**
```python
def create_task(title: str, project_id: str, due_date: Optional[str] = None) -> Task:
    """Creates a task. Validates project exists, title non-empty."""
    # Mechanical validation only
    if not title.strip():
        raise ValueError("Title required")
    if not project_exists(project_id):
        raise ValueError(f"Project {project_id} not found")
    # Create and return
    return Task.create(title=title, project_id=project_id, due_date=due_date)
```

**Bad domain tool (bundles judgment):**
```python
def smart_create_task(description: str) -> Task:
    """Parses description, decides project, sets priority, assigns due date."""
    # This bundles too much judgment - let the agent decide these things
```

---

## Files as Universal Interface

Files are the most battle-tested agent interface:

- Agents know bash (`cat`, `grep`, `mv`, `mkdir`) fluently
- Users can inspect and edit agent work directly
- State is portable, syncable, self-documenting
- No black boxes

### Why Files Over Database for Agent State

| Files | Database |
|-------|----------|
| `cat progress.md` is self-explanatory | `SELECT * FROM work_units` requires schema knowledge |
| Agents know grep, cat, sed fluently | SQL requires more cognitive overhead |
| File paths are self-documenting | Queries need context to understand |
| Humans can inspect/edit directly | Requires tooling to view/modify |

**Use files for**: Agent state, progress tracking, feature lists, session logs

**Use database for**: UI state, search indexes, cross-project queries, analytics

### Directory Conventions

```
entity_type/entity_id/
├── primary content
├── metadata.json
└── agent_log.md
```

- Use lowercase with underscores
- Markdown for human-readable content
- JSON for structured data

---

## Context File Pattern

A `CLAUDE.md` or `context.md` at project root provides session context:

```markdown
# Project Context

## What This Is
[Brief project description and purpose]

## Quick Start
[How to run/build the project]

## Available Resources
- 12 notes in /notes
- 3 active projects in /projects
- API docs in /docs/api.md

## Guidelines
- Run tests before committing
- Keep functions under 50 lines
- Use existing patterns from /src/utils

## Current State
- Feature X in progress
- Blocked on API documentation
```

This file should be:
- Read at the start of every session
- Updated when major state changes occur
- Concise—link to details rather than duplicating them
