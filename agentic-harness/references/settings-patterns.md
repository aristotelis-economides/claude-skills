# Settings Patterns for Agentic Systems

Configuration architecture patterns extracted from production agentic systems (Claude Code, etc).

## Table of Contents

- [Hierarchical Configuration](#hierarchical-configuration)
- [Permission Architecture](#permission-architecture)
- [Hook System for Agent Coordination](#hook-system-for-agent-coordination)
- [Environment Management](#environment-management)

---

## Hierarchical Configuration

### Layered Settings with Clear Precedence

```
HIGHEST PRIORITY
    ↓
1. Managed/Organizational Settings (cannot be overridden)
    ↓
2. Project Local Settings (.agent/settings.local.json - not committed)
    ↓
3. Project Team Settings (.agent/settings.json - committed)
    ↓
4. User Global Settings (~/.config/app/settings.json)
    ↓
LOWEST PRIORITY (built-in defaults)
```

**Why this matters:**
- Organizations can enforce security policies
- Teams can share standards without losing individual flexibility
- Secrets stay local, conventions stay shared
- Clear audit trail and accountability

### Settings File Structure

```json
{
  "$schema": "https://your-app.com/settings-schema.json",

  "permissions": { /* ... */ },
  "hooks": { /* ... */ },
  "env": { /* ... */ },
  "agents": { /* ... */ }
}
```

**Validation:** Provide JSON Schema for IDE autocomplete and validation.

---

## Permission Architecture

### Three-Tier Permission Model

```json
{
  "permissions": {
    "deny": [
      "Bash(rm -rf:*)",
      "Bash(sudo:*)",
      "Read(.env*)",
      "Read(./secrets/**)"
    ],
    "allow": [
      "Bash(git status)",
      "Bash(npm run test:*)",
      "Read(**/*.md)"
    ],
    "ask": [
      "Bash(git push:*)",
      "Bash(npm install:*)",
      "Write(**/*.ts)"
    ]
  }
}
```

**Evaluation order (crucial):**
1. **Deny rules** → Block immediately (highest priority)
2. **Allow rules** → Auto-approve if matched
3. **Ask rules** → Prompt user for confirmation
4. **Default mode** → Fallback behavior

**Pattern syntax:** Use glob patterns for file paths and `tool(pattern:*)` for commands.

### Permission Modes

```json
{
  "defaultMode": "default"  // "default" | "acceptEdits" | "plan" | "bypassPermissions"
}
```

| Mode | Behavior | Use Case |
|------|----------|----------|
| `default` | Prompts on first use | Normal development |
| `acceptEdits` | Auto-accepts file edits | Trusted codebases |
| `plan` | Read-only analysis | Exploration/auditing |
| `bypassPermissions` | **DANGEROUS**: Auto-accepts all | Never use in production |

**Enterprise control:**
```json
{
  "disableBypassPermissionsMode": "disable"  // Prevent dangerous mode
}
```

### Security Best Practices

1. **Deny-first**: Block dangerous operations explicitly
2. **Least privilege**: Only allow 100% safe commands
3. **Layer security**: OS-level sandboxing + permission rules
4. **Audit regularly**: Review settings monthly
5. **Test in sandbox**: Validate configs before production

---

## Hook System for Agent Coordination

### Lifecycle Hooks

Hooks run at strategic points in agent execution:

```json
{
  "hooks": {
    "PreToolUse": [/* Before tool execution */],
    "PostToolUse": [/* After successful execution */],
    "PermissionRequest": [/* When asking user */],
    "UserPromptSubmit": [/* Before processing prompt */],
    "SessionStart": [/* On session init */],
    "SessionEnd": [/* On session cleanup */]
  }
}
```

### Hook Configuration Pattern

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write(.*\\.py)",
        "hooks": [
          {
            "type": "command",
            "command": "black ${TOOL_INPUT_FILE_PATH}"
          },
          {
            "type": "command",
            "command": "git add ${TOOL_INPUT_FILE_PATH}"
          }
        ]
      }
    ],
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "validate-command.sh ${TOOL_INPUT_COMMAND}"
          }
        ]
      }
    ]
  }
}
```

### Hook Capabilities

**PreToolUse hooks can:**
- Modify tool inputs before execution
- Validate and block unsafe operations
- Add contextual metadata
- Log for auditing

**PostToolUse hooks can:**
- Run automated workflows (lint, test, format)
- Notify other agents or systems
- Update project state files
- Trigger CI/CD pipelines

**Available variables:**
- `${TOOL_INPUT_FILE_PATH}` - File being operated on
- `${TOOL_INPUT_COMMAND}` - Command being executed
- `${TOOL_NAME}` - Name of tool used
- Custom env vars from settings

### Multi-Agent Coordination via Hooks

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write(src/**/*.ts)",
        "hooks": [
          {
            "type": "command",
            "command": "notify-agent code-reviewer --file ${TOOL_INPUT_FILE_PATH}"
          }
        ]
      }
    ]
  }
}
```

**Pattern:** Use hooks to create agent pipelines without tight coupling.

---

## Environment Management

### Environment Variables in Settings

```json
{
  "env": {
    "NODE_ENV": "development",
    "API_BASE_URL": "https://api.example.com",
    "LOG_LEVEL": "info"
  }
}
```

**Scope:** Environment variables are injected when agent executes tools.

### Persistent Environment

**Method 1: ENV_FILE reference**
```json
{
  "envFile": ".agent/env.sh"
}
```

Agent sources this file before each command execution.

**Method 2: SessionStart hook**
```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "*",
        "hooks": [
          {
            "type": "command",
            "command": "source .agent/setup-env.sh"
          }
        ]
      }
    ]
  }
}
```

### Secrets Management

**Never commit secrets to team settings.**

Use project local settings for secrets:

```
.agent/
├── settings.json              # Team settings (committed)
├── settings.local.json        # Secrets (gitignored)
└── .gitignore
```

`.gitignore`:
```
settings.local.json
```

---

## Multi-Agent Settings Pattern

### Per-Agent Configuration

```json
{
  "agents": {
    "code-writer": {
      "permissions": {
        "allow": ["Read(**/*.ts)", "Write(src/**/*.ts)"],
        "deny": ["Write(package.json)"],
        "ask": ["Bash(npm install*)"]
      },
      "model": "sonnet",
      "hooks": {
        "PostToolUse": [
          {
            "matcher": "Write",
            "hooks": [
              {
                "type": "notify",
                "target": "code-reviewer"
              }
            ]
          }
        ]
      }
    },
    "code-reviewer": {
      "permissions": {
        "allow": ["Read(**/*)", "Bash(npm run lint)", "Bash(npm run test)"],
        "deny": ["Write(**/*)", "Bash(npm install*)"]
      },
      "model": "opus"
    }
  }
}
```

### Resource Management

```json
{
  "resourceLimits": {
    "maxTokensPerAgent": 100000,
    "maxConcurrentAgents": 5,
    "maxToolCallsPerMinute": 60,
    "timeoutSeconds": 300
  }
}
```

### Error Handling

```json
{
  "errorHandling": {
    "retryPolicy": {
      "maxRetries": 3,
      "backoffMultiplier": 2,
      "initialDelayMs": 1000
    },
    "fallbackModel": "haiku",
    "escalationRules": [
      {
        "condition": "toolCallFailed",
        "action": "notifyHuman",
        "threshold": 3
      }
    ]
  }
}
```

---

## Key Principles

1. **Security by Default**: Deny-first permission model
2. **Hierarchical Control**: Organizational policy → Team standards → Individual preferences
3. **Event-Driven Coordination**: Hooks for agent-to-agent communication
4. **Declarative over Imperative**: JSON config instead of code
5. **Fail-Safe Mechanisms**: Escalation policies, retry logic, human-in-the-loop
6. **Separation of Concerns**: Secrets local, conventions shared, policies enforced
7. **Version Control Friendly**: Team settings committed, local settings ignored
8. **Observable**: Built-in audit logging, telemetry, debugging support

---

## Example: Complete Settings File

```json
{
  "$schema": "https://your-app.com/settings-schema.json",

  "permissions": {
    "deny": [
      "Bash(rm -rf:*)",
      "Bash(sudo:*)",
      "Read(.env*)"
    ],
    "allow": [
      "Bash(git status)",
      "Read(**/*.md)"
    ],
    "ask": [
      "Bash(git push:*)",
      "Write(**/*.ts)"
    ]
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
    ],
    "SessionStart": [
      {
        "matcher": "*",
        "hooks": [
          {
            "type": "command",
            "command": "./init.sh"
          }
        ]
      }
    ]
  },

  "env": {
    "NODE_ENV": "development",
    "LOG_LEVEL": "info"
  },

  "agents": {
    "code-writer": {
      "permissions": {
        "allow": ["Write(src/**/*.ts)"]
      },
      "model": "sonnet"
    }
  },

  "resourceLimits": {
    "maxConcurrentAgents": 5,
    "timeoutSeconds": 300
  }
}
```
