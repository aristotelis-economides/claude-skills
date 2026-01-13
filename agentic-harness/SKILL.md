---
name: agentic-harness
description: Design principles for building effective agentic systems and agent-native applications. Use when initializing CLAUDE.md files, designing agent architectures, creating multi-context-window workflows, building applications where agents are first-class citizens, or setting up long-running autonomous coding projects.
---

# Agentic Harness Design

Principles for building systems where agents work effectively across sessions, handle complex tasks autonomously, and leave clean state for continuation.

## Core Architecture

### Two-Agent Pattern for Long-Running Tasks

Complex projects spanning multiple context windows require distinct phases:

**Initializer Agent (first context window only):**
- Analyzes requirements and creates comprehensive feature list
- Sets up environment scaffolding (init.sh, progress tracking, git repo)
- Writes structured tests/specifications in JSON format
- Creates initial commit as baseline

**Working Agent (all subsequent windows):**
- Reads progress files and git history to orient
- Selects ONE feature/task to complete
- Makes incremental progress with commits
- Leaves environment in clean, documented state

### The Agent-Native Principles

1. **Parity**: Agent can achieve anything users can through the UI via tools
2. **Granularity**: Tools are atomic primitives; features are outcomes achieved by agents in loops
3. **Composability**: New features emerge from combining atomic tools via prompts
4. **Emergent Capability**: Agent handles tasks you didn't explicitly design for

## State Management

### Required Artifacts

Create these in the first context window:

```
project/
├── init.sh              # Environment setup, server start, basic tests
├── progress.md          # Freeform notes on what's been done
├── features.json        # Structured feature list with pass/fail status
└── .git/                # Version control for checkpoints and history
```

### Feature List Format (JSON, not Markdown)

Use JSON to prevent accidental modification of test content:

```json
{
  "features": [
    {
      "id": "auth-001",
      "category": "authentication",
      "description": "User can log in with email and password",
      "steps": [
        "Navigate to login page",
        "Enter valid credentials",
        "Verify redirect to dashboard"
      ],
      "passes": false
    }
  ]
}
```

Instruct the agent: "Only modify the `passes` field. Never remove or edit test descriptions."

### Progress Tracking

The progress file serves as working memory across sessions:

```markdown
# Progress

## Current State
- Authentication complete and tested
- Dashboard layout implemented
- API integration in progress

## Last Session
- Completed: User login flow
- Committed: abc123 "feat: implement login with session management"

## Next Priority
- Complete API data fetching
- Add error handling for network failures

## Known Issues
- Rate limiting not yet implemented
- Mobile styles need attention
```

### Git as State Checkpoint

Git provides recoverable history:
- Commit after each completed feature
- Use descriptive messages: `feat: implement [feature]` or `fix: resolve [issue]`
- Agent can `git diff` to see recent changes
- Agent can `git revert` to recover from mistakes

## Session Lifecycle

### Starting a New Session

Agent should always begin with orientation:

```
1. pwd                                    # Confirm working directory
2. cat progress.md                        # Read what's been done
3. cat features.json | jq '.features[] | select(.passes==false) | .id' | head -5
4. git log --oneline -10                  # Review recent commits
5. ./init.sh                              # Start environment
6. [Run basic integration test]           # Verify nothing is broken
7. Select ONE incomplete feature to work on
```

### Ending a Session

Before context ends or task completes:

```
1. Ensure all changes compile/run without errors
2. git add -A && git commit -m "descriptive message"
3. Update progress.md with what was done
4. Update features.json passes field if feature complete
5. Note any blockers or next steps in progress.md
```

### Context Window Management

- **Start fresh over compacting** when possible—agents discover state well from filesystem
- **Checkpoint frequently** if context may be interrupted
- **Don't stop early** due to context concerns; save state and let next session continue

## Tool Design

### Atomic Over Bundled

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

### Tool Design Principles

- **Atomic primitives first**: Start with bash, file operations, basic storage
- **Domain tools for vocabulary**: `create_note` teaches what "note" means in your system
- **Keep primitives available**: Domain tools are shortcuts, not gates
- **CRUD completeness**: Every entity needs create, read, update, delete operations

### When to Add Domain Tools

Add domain-specific tools when:
- Establishing vocabulary (what is a "note", "task", "project" in this system)
- Adding guardrails for destructive operations
- Optimizing hot paths for speed/cost
- Preventing common errors through validation

Domain tools should represent ONE conceptual user action. Include mechanical validation, but judgment about *what* to do belongs in prompts.

## Files as Universal Interface

Files are the most battle-tested agent interface:
- Agents know bash (`cat`, `grep`, `mv`, `mkdir`) fluently
- Users can inspect and edit agent work directly
- State is portable, syncable, self-documenting
- No black boxes

### Directory Conventions

```
entity_type/entity_id/
├── primary content
├── metadata.json
└── agent_log.md
```

Use lowercase with underscores. Markdown for human-readable content; JSON for structured data.

### Context File Pattern

A context.md or CLAUDE.md file at project root provides session context:

```markdown
# Context

## What This Is
[Project description and purpose]

## Available Resources
- 12 notes in /notes
- 3 active projects in /projects

## Recent Activity
- Completed authentication (2 hours ago)
- Fixed login bug (yesterday)

## Guidelines
- Run tests before committing
- Keep functions under 50 lines
- Use existing patterns from /src/utils

## Current State
- Feature X in progress
- Blocked on API documentation
```

## Testing and Verification

### Self-Verification is Critical

Agents must verify their own work:
- Unit tests alone are insufficient
- End-to-end testing catches integration issues
- Browser automation (Puppeteer, Playwright) for UI verification
- Always test as a user would before marking complete

### Verification Prompt Pattern

```
After implementing a feature:
1. Run the relevant test suite
2. Manually verify the feature end-to-end as a user would
3. Only mark as complete after successful verification
4. If verification fails, fix and re-verify before continuing
```

## Multi-Agent Considerations

### Subagent Orchestration

Claude can delegate to subagents when beneficial. Ensure:
- Subagent tools are well-defined with clear scope
- Let orchestrating agent decide when to delegate
- Subagent results should be verifiable

### Parallel Tool Execution

Maximize parallel tool calls when operations are independent:
- Reading multiple files simultaneously
- Running independent bash commands
- Searching multiple sources

Sequential execution when dependencies exist between calls.

## Anti-Patterns

### Avoid These Patterns

**Agent as router**: Using agent just to route to functions removes judgment
**Request/response thinking**: Missing the loop—agent should pursue outcomes until complete
**Workflow-shaped tools**: `analyze_and_organize()` bundles judgment; break into primitives
**Orphan UI actions**: Users can do things agent cannot achieve
**Context starvation**: Agent doesn't know what exists or what's been done
**Heuristic completion**: Detect completion explicitly, not through heuristics
**Happy path in code**: Let agent handle edge cases with judgment

### Failure Modes to Prevent

1. **One-shotting**: Trying to complete everything at once
   - Fix: Enforce incremental, feature-by-feature progress

2. **Premature completion**: Declaring done without verification
   - Fix: Require end-to-end testing before marking complete

3. **Context loss**: New session doesn't know what happened
   - Fix: Progress files, git history, structured feature tracking

4. **Unclean state**: Leaving bugs, half-implemented features
   - Fix: Commit only working code, document known issues

5. **Test gaming**: Hard-coding to pass specific tests
   - Fix: Emphasize general solutions, verify with varied inputs

## Prompt Patterns

### For Initializer Agent

```
You are setting up a new project. Your job is to:
1. Analyze the requirements and create a comprehensive feature list in features.json
2. Create init.sh to set up the environment and run basic tests
3. Create progress.md for tracking work across sessions
4. Make an initial git commit as a baseline

Write features as testable specifications. Mark all as passes: false initially.
Future sessions will implement these one at a time.
```

### For Working Agent

```
This is a continuation of ongoing work. Start by:
1. Read progress.md and recent git history
2. Run init.sh to start the environment
3. Run a basic integration test to verify nothing is broken
4. Select ONE incomplete feature from features.json

Work on that single feature until complete. Before ending:
- Ensure code works without errors
- Commit with descriptive message
- Update progress.md with what was done
- Update features.json if feature is complete
```

### For Maximum Persistence

```
Your context window will be automatically compacted as it approaches its limit.
Do not stop tasks early due to context concerns. Save progress and continue.
Be as persistent and autonomous as possible. Complete features fully.
If you must stop, ensure clean state: committed code, updated progress, clear notes.
```
