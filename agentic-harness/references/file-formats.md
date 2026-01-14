# File Format Reference

Detailed templates and examples for agentic harness state files.

## Table of Contents

- [sessions.md](#sessionsmd)
- [progress.md](#progressmd)
- [features.json](#featuresjson)
- [decisions.json](#decisionsjson)
- [settings.json](#settingsjson)
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

## decisions.json

Logs human decisions for runtime learning. See [references/runtime-patterns.md](runtime-patterns.md#decision-logging) for detailed usage.

```json
{
  "decisions": [
    {
      "timestamp": "2025-01-13T14:32:00Z",
      "context": "Agent asked whether to refactor auth module or add new feature",
      "options_presented": ["Refactor first", "Add feature first", "Do both in parallel"],
      "user_choice": "Refactor first",
      "reasoning": "User mentioned tech debt was slowing them down",
      "outcome": "Refactor completed, feature added more easily afterward"
    },
    {
      "timestamp": "2025-01-12T09:15:00Z",
      "context": "Agent proposed two API designs",
      "options_presented": ["REST endpoints", "GraphQL schema"],
      "user_choice": "REST endpoints",
      "reasoning": "Team more familiar with REST",
      "outcome": null
    }
  ]
}
```

**Purpose:** Over time, patterns emerge that allow agents to pre-select likely preferences and reduce decision fatigue.

---

## settings.json

Configuration for permissions, hooks, environment, and agent behavior. See [references/settings-patterns.md](settings-patterns.md) for comprehensive guide.

### Minimal Example

```json
{
  "$schema": "https://your-app.com/settings-schema.json",

  "permissions": {
    "deny": ["Bash(rm -rf:*)", "Read(.env*)"],
    "allow": ["Bash(git status)", "Read(**/*.md)"],
    "ask": ["Bash(git push:*)", "Write(**/*.ts)"]
  },

  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write(.*\\.py)",
        "hooks": [
          {
            "type": "command",
            "command": "black ${TOOL_INPUT_FILE_PATH}"
          }
        ]
      }
    ]
  },

  "env": {
    "NODE_ENV": "development"
  }
}
```

### Settings Hierarchy

```
1. .agent/settings.json          (Team - committed)
2. .agent/settings.local.json    (Personal - gitignored)
3. ~/.config/app/settings.json   (User global)
```

Higher priority settings override lower ones.

---

## Directory Structures

### 1. Source Code Project Structure (Development Time)

**When building your agentic application with a coding agent**, keep agent state files in `.agent/`:

```
my-agentic-app-source/         # Your source code repo
├── .agent/                    # Coding agent's state (for building the app)
│   ├── sessions.md            # Session log
│   ├── progress.md            # Project state snapshot
│   ├── features.json          # Feature tracking
│   └── .gitignore             # Ignore local agent state
│
├── src/                       # Your application source code
│   ├── cli.ts                 # CLI entry point
│   ├── agent/                 # Agent orchestration logic
│   ├── tools/                 # Tool implementations
│   └── config/                # Config loading logic
│
├── init.sh                    # Environment setup for development
├── package.json
└── .git/
```

**`.agent/.gitignore`:**
```
*.log
*.tmp
```

**Purpose:** This is where the agent that's *building your application* tracks its progress. Think of it like Claude Code's session tracking while developing your app.

---

### 2. Installed Application Structure (Runtime)

**When users run your built agentic application**, it should store settings and runtime state in standard locations:

#### User-Global Settings (Application Config)

```
~/.config/<app-name>/          # Linux/macOS
├── settings.json              # User's global settings
├── cache/                     # Application cache
└── logs/                      # Application logs

# or on macOS:
~/Library/Application Support/<app-name>/
├── settings.json
└── ...

# or on Windows:
C:\Users\<username>\AppData\Roaming\<app-name>\
├── settings.json
└── ...
```

**`settings.json` (user global):**
```json
{
  "permissions": {
    "deny": ["Bash(rm -rf:*)", "Read(.env*)"],
    "allow": ["Bash(git status)"]
  },
  "env": {
    "LOG_LEVEL": "info"
  }
}
```

**Purpose:** User's personal preferences that apply to all projects.

#### Per-Project Settings (Where User Works)

```
~/users-work-project/          # User's actual work (NOT your source code)
├── .myapp/                    # Your app's project-specific state
│   ├── settings.json          # Project team settings (committed)
│   ├── settings.local.json    # User's local overrides (gitignored)
│   ├── sessions.md            # Session history (if applicable)
│   ├── decisions.json         # Decision log for learning
│   └── .gitignore             # Ignore local settings
│
├── ... (user's actual project files)
└── .git/
```

**`.myapp/settings.json` (project team settings):**
```json
{
  "permissions": {
    "deny": ["Write(secrets/**)"],
    "allow": ["Bash(npm run test)"]
  },
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write(.*\\.ts)",
        "hooks": [{"type": "command", "command": "npm run lint"}]
      }
    ]
  }
}
```

**`.myapp/settings.local.json` (user personal overrides):**
```json
{
  "env": {
    "GITHUB_TOKEN": "ghp_..."
  }
}
```

**`.myapp/.gitignore`:**
```
settings.local.json
*.log
*.tmp
sessions.md
decisions.json
```

**Purpose:** Project-specific configuration and runtime state when your app is running on a user's project.

#### System-Wide Managed Settings (Enterprise/Organizational)

```
# Linux
/etc/<app-name>/
├── managed-settings.json

# macOS
/Library/Application Support/<app-name>/
├── managed-settings.json

# Windows
C:\ProgramData\<app-name>\
├── managed-settings.json
```

**Purpose:** Organization-enforced policies that cannot be overridden by users.

---

### Settings Resolution Order (Runtime)

When your agentic application runs, it loads settings in this order (highest priority first):

```
1. /etc/<app-name>/managed-settings.json           (Organizational - cannot override)
    ↓
2. ~/project/.myapp/settings.local.json            (User's project-local secrets)
    ↓
3. ~/project/.myapp/settings.json                  (Team's project settings)
    ↓
4. ~/.config/<app-name>/settings.json              (User's global preferences)
    ↓
5. Built-in defaults                                (Hardcoded in your app)
```

---

### Summary

| Context | Location | Purpose | Committed to Git? |
|---------|----------|---------|-------------------|
| **Development** | `my-app-source/.agent/` | Coding agent's state while building your app | No (agent workspace) |
| **Runtime: User Global** | `~/.config/<app-name>/` | User's personal app preferences | N/A (not in repo) |
| **Runtime: Project Team** | `~/project/.myapp/settings.json` | Team's shared project config | Yes |
| **Runtime: Project Local** | `~/project/.myapp/settings.local.json` | User's project-specific secrets | No (gitignored) |
| **Runtime: Organizational** | `/etc/<app-name>/` | Enterprise policies | N/A (system-wide) |

**Key Insight:** `.agent/` is for *building* your app. `~/.config/<app-name>/` and `.myapp/` are for *running* your app.
