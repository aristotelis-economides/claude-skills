# Templates and Examples

Practical scaffolding for agentic projects. Copy and adapt these to your needs.

For file format templates (sessions.md, progress.md, features.json), see [file-formats.md](file-formats.md).

## Table of Contents

- [CLAUDE.md Template](#claudemd-template)
- [init.sh Template](#initsh-template)
- [System Prompts](#system-prompts)
- [Approval Patterns](#approval-patterns)
- [Completion Signals](#completion-signals)

---

## CLAUDE.md Template

```markdown
# Project: [Name]

## Overview
[One paragraph describing what this project does]

## Tech Stack
- [Language/framework]
- [Key dependencies]

## Directory Structure
```
src/           # Source code
tests/         # Test files
docs/          # Documentation
.agent/        # Agent state files
```

## Development

### Setup
```bash
./init.sh
```

### Running
```bash
[command to run]
```

### Testing
```bash
[command to test]
```

## Working Conventions

### Before Making Changes
- Read .agent/sessions.md for recent session history
- Read .agent/progress.md for current state
- Check .agent/features.json for priorities
- Run basic integration test

### Code Style
- [Style guidelines]
- [Naming conventions]
- [Pattern preferences]

### After Making Changes
- Run tests
- Commit with descriptive message
- Add session entry to TOP of .agent/sessions.md
- Update .agent/progress.md
- Update .agent/features.json if feature complete

## Known Issues
[List of known bugs or limitations]
```

---

## init.sh Template

```bash
#!/bin/bash
set -e

echo "=== Project Initialization ==="

# Navigate to project root
cd "$(dirname "$0")"

# Install dependencies
echo "Installing dependencies..."
# npm install / pip install -r requirements.txt / etc.

# Set up environment
echo "Setting up environment..."
# cp .env.example .env (if needed)

# Start services
echo "Starting services..."
# docker-compose up -d / start dev server / etc.

# Wait for services
echo "Waiting for services to be ready..."
sleep 2

# Basic health check
echo "Running health check..."
# curl -f http://localhost:3000/health || exit 1

echo "=== Initialization Complete ==="
echo "Ready for development"
```

---

## System Prompts

### Initializer Agent

```
You are initializing a new project for autonomous development. Set up scaffolding that enables productive work across many future sessions.

Create the following:

1. **CLAUDE.md** - Project context:
   - Project overview and purpose
   - Tech stack and dependencies
   - Directory structure
   - Development commands (setup, run, test)
   - Coding conventions

2. **.agent/features.json** - Feature specifications:
   - Break requirements into testable features
   - Each feature: id, description, testSteps, passes (boolean)
   - Mark all as passes: false initially
   - Order by dependency (foundational first)

3. **.agent/sessions.md** - Session log:
   - HOW TO READ and HOW TO UPDATE instructions at top
   - Empty session log (will be filled by working agents)

4. **.agent/progress.md** - Project state:
   - Current phase
   - Completed work section (empty initially)
   - Next steps
   - Known issues

5. **init.sh** - Environment setup:
   - Install dependencies
   - Start required services
   - Run basic health check

After creating files:
- Initialize git repository
- Make initial commit: "chore: initialize project scaffolding"
```

### Working Agent

```
You are continuing work on an ongoing project.

## Starting a Session

1. Orient:
   - Read .agent/sessions.md (head -50) for recent history
   - Read .agent/progress.md for current state
   - Check incomplete features in .agent/features.json
   - git log --oneline -10 for recent commits

2. Verify environment:
   - Run ./init.sh
   - Run basic integration test
   - If broken, fix before proceeding

3. Select ONE feature where passes: false

## During Work

- Commit incrementally as you complete logical units
- Run tests frequently
- If feature is complex, break into sub-tasks

## Ending a Session

1. Ensure all changes compile/run without errors
2. git add -A && git commit -m "descriptive message"
3. Add session block to TOP of .agent/sessions.md
4. Update .agent/progress.md
5. Set passes: true in .agent/features.json (only modify passes field, never edit tests)

## Auto-Continue

After completing a feature:
- If incomplete features remain AND no blockers → continue to next feature
- Do NOT ask "should I continue?" — just continue

## Stop When

- All features pass
- Blocker requiring human input
- Tests fail repeatedly and cannot be fixed
- Need requirement clarification
```

---

## Approval Patterns

Match approval requirements to risk and reversibility:

| Stakes | Reversibility | Pattern | Example |
|--------|---------------|---------|---------|
| Low | Easy | Auto-apply | Organizing files |
| Low | Hard | Quick confirm | Publishing content |
| High | Easy | Suggest + apply | Code refactoring |
| High | Hard | Explicit approval | Sending emails, payments |

---

## Completion Signals

Agent should explicitly signal completion rather than relying on heuristics:

```typescript
interface ToolResult {
  success: boolean;
  output: string;
  shouldContinue: boolean;  // false = loop complete
}

// Success and continue
return { success: true, output: "Created file", shouldContinue: true };

// Failure but continue (will retry)
return { success: false, output: "Network error", shouldContinue: true };

// Complete - stop the loop
return { success: true, output: "All done", shouldContinue: false };
```
