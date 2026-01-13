# Templates and Examples

Practical scaffolding for agentic projects. Copy and adapt these to your needs.

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
- Read progress.md for current state
- Check features.json for priorities
- Run basic integration test

### Code Style
- [Style guidelines]
- [Naming conventions]
- [Pattern preferences]

### After Making Changes
- Run tests
- Commit with descriptive message
- Update progress.md
- Update features.json if feature complete

## Current State
[Updated by working agent each session]

## Known Issues
[List of known bugs or limitations]
```

## features.json Template

```json
{
  "project": "project-name",
  "created": "2025-01-01",
  "features": [
    {
      "id": "core-001",
      "category": "core",
      "priority": 1,
      "description": "Basic project setup and configuration",
      "acceptance_criteria": [
        "Project structure created",
        "Dependencies installed",
        "Basic configuration works"
      ],
      "passes": false,
      "notes": ""
    },
    {
      "id": "auth-001",
      "category": "authentication",
      "priority": 2,
      "description": "User can create account and log in",
      "acceptance_criteria": [
        "Registration form works",
        "Login form works",
        "Session persists across page reloads",
        "Logout clears session"
      ],
      "passes": false,
      "notes": ""
    }
  ],
  "meta": {
    "total": 2,
    "passing": 0,
    "failing": 2
  }
}
```

## progress.md Template

```markdown
# Progress Log

## Project Status
- **Started**: [date]
- **Last Updated**: [date]
- **Features Complete**: 0/N

## Current Session
### Working On
[What the current session is focused on]

### Blockers
[Any issues preventing progress]

## Session History

### Session N (date)
- **Completed**: [list of completed items]
- **Commits**: [commit hashes and messages]
- **Notes**: [any relevant observations]

## Next Steps
1. [Priority item 1]
2. [Priority item 2]
3. [Priority item 3]

## Architecture Decisions
- [Decision 1]: [Rationale]
- [Decision 2]: [Rationale]

## Known Issues
- [ ] [Issue description]
- [ ] [Issue description]
```

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

## System Prompt: Initializer Agent

```
You are initializing a new project for autonomous development. Your task is to set up scaffolding that enables productive work across many future sessions.

Create the following files:

1. **CLAUDE.md** - Project context file with:
   - Project overview and purpose
   - Tech stack and dependencies
   - Directory structure explanation
   - Development commands (setup, run, test)
   - Coding conventions and patterns
   - Current state section (to be updated each session)

2. **features.json** - Structured feature specifications:
   - Break requirements into testable features
   - Each feature has: id, category, description, acceptance_criteria, passes (boolean)
   - Be comprehensive - 20-50+ features for substantial projects
   - Mark all as passes: false initially
   - Order by dependency (foundational features first)

3. **progress.md** - Session tracking:
   - Current status overview
   - Session history template
   - Next steps section
   - Known issues tracker

4. **init.sh** - Environment setup script:
   - Install dependencies
   - Start required services
   - Run basic health check
   - Print ready status

After creating these files:
- Initialize git repository
- Make initial commit: "chore: initialize project scaffolding"
- Report summary of what was created
```

## System Prompt: Working Agent

```
You are continuing work on an ongoing project. This is session N of a multi-session task.

## Starting a Session

1. Orient yourself:
   - `pwd` to confirm location
   - Read CLAUDE.md for project context
   - Read progress.md for recent work
   - `git log --oneline -10` for recent commits
   - Check features.json for incomplete features

2. Verify environment:
   - Run `./init.sh` to start services
   - Run a basic integration test to verify nothing is broken
   - If broken, fix before proceeding

3. Select ONE feature:
   - Choose highest priority incomplete feature
   - Note your selection in progress.md

## During Work

- Make incremental commits as you complete logical units
- If you encounter issues, document in progress.md
- Run tests frequently to catch regressions
- If a feature is more complex than expected, break it into sub-tasks

## Ending a Session

Before your context ends:
1. Ensure all changes compile/run without errors
2. `git add -A && git commit -m "descriptive message"`
3. Update progress.md:
   - What was completed
   - What remains
   - Any blockers or issues
4. Update features.json if feature passes all acceptance criteria
5. Push if remote is configured

## Important Rules

- Never remove or edit feature descriptions in features.json—only modify `passes` and `notes`
- Always verify features end-to-end before marking complete
- If interrupted, leave environment in working state
- Be specific in progress notes—next session has no memory of this one
```

## Approval Patterns by Risk Level

| Stakes | Reversibility | Pattern | Example |
|--------|---------------|---------|---------|
| Low | Easy | Auto-apply | Organizing files |
| Low | Hard | Quick confirm | Publishing content |
| High | Easy | Suggest + apply | Code refactoring |
| High | Hard | Explicit approval | Sending emails, payments |

## Completion Signal Pattern

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
