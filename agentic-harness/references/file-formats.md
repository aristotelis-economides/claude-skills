# File Format Reference

Detailed templates and examples for agentic harness state files.

## Table of Contents

- [sessions.md](#sessionsmd)
- [progress.md](#progressmd)
- [features.json](#featuresjson)
- [Per-Project Directory Structure](#per-project-directory-structure)

---

## sessions.md

Chronological session log, newest first. Enables any new agent to quickly understand what happened and continue.

```markdown
# Session Log

## HOW TO READ THIS FILE
- Each session is separated by `=== Session N (ISO-TIMESTAMP) ===`
- "Completed" lists features done this session with their ID from features.json
- "Next" is what the next session should tackle FIRST
- "Blockers" are issues preventing progress
- "Notes" are context for future sessions

## HOW TO UPDATE THIS FILE
- Add new session block at the TOP of the file (newest first)
- Use ISO 8601 timestamps: YYYY-MM-DDTHH:MM:SSZ
- Reference feature IDs from features.json
- Be specific about what was done and what's next

---

=== Session 3 (2025-01-13T14:32:00Z) ===
Completed:
- [auth-003] User login flow with email/password validation
- [auth-004] Session management with JWT tokens

Commits:
- abc123 "feat: implement login with session management"

Blockers: None

Next:
- [api-001] API data fetching
- [api-002] Error handling for network failures

Notes:
- Chose JWT over session cookies for stateless auth
- Login form includes "remember me" checkbox

=== Session 2 (2025-01-12T09:15:00Z) ===
Completed:
- [auth-001] Auth module structure
- [auth-002] Password hashing utilities

Commits:
- def456 "feat: add auth module scaffold"

Blockers:
- Had to switch from bcrypt to argon2 due to native dependency issues

Next:
- [auth-003] Implement login form
- [auth-004] Session management

Notes:
- argon2 works better cross-platform than bcrypt

=== Session 1 (2025-01-11T10:00:00Z) ===
Completed:
- Project initialization
- Created features.json with Phase 1 features
- Set up git repository

Commits:
- 123abc "chore: initial project setup"

Blockers: None

Next:
- [auth-001] Begin auth module

Notes:
- Initial session, scaffolding complete
```

---

## progress.md

Living snapshot of current project state. Gets *updated* (not appended to) as the project evolves.

```markdown
# Project Progress

## Last Updated
2025-01-13

## Current Phase
Phase 1: Foundation (MVP)

## Completed Work

### Phase 1 Tasks Completed
- [x] User authentication with JWT (auth-001 through auth-004)
- [x] Password hashing with argon2

### Technical Implementation
- **Auth**: JWT-based stateless authentication
- **Security**: argon2 for password hashing (chosen over bcrypt for cross-platform compatibility)

## Current State
- App builds and runs successfully
- Users can register and log in
- Session management working with "remember me" option

## Next Steps
1. Start API data fetching (api-001)
2. Add error handling for network failures (api-002)

## Known Issues
- Rate limiting not yet implemented
- Mobile styles need attention

## Architecture Decisions
- Using JWT for stateless auth (see commit abc123)
- Chose argon2 over bcrypt for password hashing
```

---

## features.json

Flat feature list with boolean `passes` field. Use JSON to prevent accidental modification of test content.

**Critical rule**: Only modify the `passes` field. Never delete or edit feature descriptions or test steps.

```json
{
  "features": [
    {
      "id": "auth-001",
      "description": "User can create account with email and password",
      "testSteps": [
        "Navigate to registration page",
        "Enter valid email and password",
        "Submit form",
        "Verify account created and user logged in"
      ],
      "passes": false
    },
    {
      "id": "auth-002",
      "description": "User can log in with existing credentials",
      "testSteps": [
        "Navigate to login page",
        "Enter valid credentials",
        "Submit form",
        "Verify redirect to dashboard"
      ],
      "passes": false
    },
    {
      "id": "api-001",
      "description": "App fetches and displays data from external API",
      "testSteps": [
        "Trigger data fetch",
        "Verify loading state shown",
        "Verify data displayed correctly",
        "Verify error handling on failure"
      ],
      "passes": false
    }
  ]
}
```

**Why all start as `passes: false`**: Prevents premature task completion. Agent must verify each feature end-to-end before marking it passing.

### Querying features.json

```bash
# List incomplete features
cat features.json | jq '.features[] | select(.passes == false) | .id'

# Count passing vs failing
cat features.json | jq '.features | {total: length, passing: map(select(.passes)) | length}'

# Get next feature to work on
cat features.json | jq '.features[] | select(.passes == false) | .id' | head -1
```

---

## Per-Project Directory Structure

Keep agent state files separate from project files using a dedicated directory:

```
user-project/
├── .agent/                    # Agent state (or .nexus/, .claude/, etc.)
│   ├── sessions.md            # Session log
│   ├── progress.md            # Project state snapshot
│   ├── features.json          # Feature tracking
│   ├── decisions.json         # Human decision log (for runtime learning)
│   └── sessions/              # Per-session debug logs (optional)
│       ├── session-001.json
│       └── session-002.json
├── init.sh                    # Environment setup script
├── ... (user's project files)
└── .git/
```

This separation:
- Keeps agent files out of the user's way
- Makes it easy to gitignore agent state if desired
- Provides clear boundaries between project code and agent metadata
