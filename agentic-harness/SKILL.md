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
- Sets up environment scaffolding (init.sh, state files, git repo)
- Writes structured tests/specifications in JSON format
- Creates initial commit as baseline

**Working Agent (all subsequent windows):**
- Reads session log and progress to orient
- Selects ONE feature/task to complete
- Makes incremental progress with commits
- Leaves environment in clean, documented state

### Agent-Native Principles

1. **Parity**: Agent can achieve anything users can through the UI via tools
2. **Granularity**: Tools are atomic primitives; features are outcomes achieved by agents in loops
3. **Composability**: New features emerge from combining atomic tools via prompts
4. **Emergent Capability**: Agent handles tasks you didn't explicitly design for

## State Management

### Required Artifacts

```
project/
├── init.sh              # Environment setup, server start, basic tests
├── progress.md          # Living snapshot of current project state
├── sessions.md          # Chronological session log (newest first)
├── features.json        # Structured feature list with phases
└── .git/                # Version control for checkpoints
```

For detailed file templates and examples, see [references/file-formats.md](references/file-formats.md).

### Git as State Checkpoint

- Commit after each completed feature
- Use descriptive messages: `feat: implement [feature]` or `fix: resolve [issue]`
- Agent can `git diff` to see recent changes
- Agent can `git revert` to recover from mistakes

## Session Lifecycle

### Starting a New Session

```
1. pwd                                    # Confirm working directory
2. head -50 sessions.md                   # Read recent session history
3. cat progress.md                        # Understand current project state
4. cat features.json | jq '.features[] | select(.passes == false) | .id' | head -5
5. git log --oneline -10                  # Review recent commits
6. ./init.sh                              # Start environment
7. [Run basic integration test]           # Verify nothing is broken
8. Select ONE incomplete feature to work on
```

### Ending a Session

```
1. Ensure all changes compile/run without errors
2. git add -A && git commit -m "descriptive message"
3. Add new session block to TOP of sessions.md
4. Update progress.md to reflect current state
5. Set passes: true in features.json if feature verified
```

### Auto-Continue Behavior

After completing each feature, automatically:

1. Run verification (typecheck, build, tests)
2. Commit with descriptive message
3. Update sessions.md and progress.md
4. Set passes: true in features.json
5. **If incomplete features remain AND no blockers → continue to next feature**

**Only stop when:**
- All features pass
- Hit a blocker requiring human input
- Tests fail repeatedly and you cannot fix them
- Need clarification on requirements

**Do NOT ask "should I continue?"—just continue if conditions allow.**

### State Recovery

When finding uncommitted changes or unclear state:

1. `git status` and `git diff` to see what's changed
2. `head -30 sessions.md` to see last known state
3. Run typecheck/build to assess what works
4. Either: complete in-progress work, or `git checkout .` to revert
5. Update sessions.md to reflect actual state

## Testing and Verification

Agents must verify their own work:

- Unit tests alone are insufficient
- End-to-end testing catches integration issues
- Browser automation (Puppeteer, Playwright) for UI verification
- Always test as a user would before marking complete

```
After implementing a feature:
1. Run the relevant test suite
2. Verify the feature end-to-end as a user would
3. Only mark complete after successful verification
4. If verification fails, fix and re-verify before continuing
```

## Multi-Agent Considerations

### Subagent Orchestration

- Subagent tools should have well-defined scope
- Let orchestrating agent decide when to delegate
- Subagent results should be verifiable

### Parallel Tool Execution

Maximize parallel tool calls when operations are independent (reading multiple files, running independent commands). Sequential execution when dependencies exist.

## Anti-Patterns and Failure Modes

**Avoid:**
- **Agent as router**: Using agent just to route to functions removes judgment
- **Request/response thinking**: Agent should pursue outcomes until complete
- **Workflow-shaped tools**: `analyze_and_organize()` bundles judgment; use primitives
- **Context starvation**: Agent doesn't know what exists or what's been done

**Prevent:**
1. **One-shotting** → Enforce incremental, feature-by-feature progress
2. **Premature completion** → Require end-to-end testing before marking complete
3. **Context loss** → Progress files, git history, structured feature tracking
4. **Unclean state** → Commit only working code, document known issues
5. **Test gaming** → Emphasize general solutions, verify with varied inputs

## Prompt Patterns

### For Initializer Agent

```
You are setting up a new project. Your job is to:
1. Analyze requirements and create comprehensive feature list in features.json
2. Create init.sh for environment setup and basic tests
3. Create sessions.md for logging session history
4. Create progress.md for tracking overall project state
5. Make initial git commit as baseline

Write features as testable specifications. Mark all as passes: false.
Future sessions will implement these one at a time.
```

### For Working Agent

```
This is a continuation of ongoing work. Start by:
1. Read sessions.md (head -50) for recent session history
2. Read progress.md for current project state
3. Run init.sh to start the environment
4. Run basic integration test to verify nothing is broken
5. Select ONE feature where passes: false

Work on that single feature until complete. Before ending:
- Ensure code works without errors
- Commit with descriptive message
- Add new session block to TOP of sessions.md
- Update progress.md to reflect current state
- Set passes: true in features.json (only modify passes field, never edit tests)
- If incomplete features remain and no blockers, continue to next feature
```

### For Maximum Persistence

```
Your context window will be automatically compacted as it approaches its limit.
Do not stop tasks early due to context concerns. Save progress and continue.
Be as persistent and autonomous as possible. Complete features fully.
If you must stop, ensure clean state: committed code, updated progress, clear notes.
```

## Additional References

- **File formats** (sessions.md, progress.md, features.json): [references/file-formats.md](references/file-formats.md)
- **Project templates** (CLAUDE.md, init.sh, system prompts): [references/templates.md](references/templates.md)
- **Tool design principles**: [references/tool-design.md](references/tool-design.md)
- **Runtime patterns** (decision logging, observe→formalize, prompt-native dev): [references/runtime-patterns.md](references/runtime-patterns.md)
