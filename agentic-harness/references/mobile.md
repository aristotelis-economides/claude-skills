# Mobile Agent Considerations

Agent-native patterns for iOS and mobile platforms. Read this when building mobile agent applications.

## The Mobile Challenge

Agents are long-running. Mobile apps are not.

An agent might need 30 seconds, 5 minutes, or an hour. But iOS backgrounds apps after seconds of inactivity and may kill them to reclaim memory. Users switch apps, take calls, or lock phones mid-task.

Mobile agent apps need:
- **Checkpointing**: Save state so work isn't lost
- **Resuming**: Pick up where you left off after interruption
- **Background execution**: Use limited iOS background time wisely

## Checkpoint and Resume

### What to Checkpoint

```swift
struct AgentCheckpoint: Codable {
    let agentType: String
    let messages: [[String: Any]]  // Conversation history
    let iterationCount: Int
    let taskListJSON: String?
    let customState: [String: String]
    let timestamp: Date
}
```

### When to Checkpoint

- On app backgrounding
- After each tool result
- Periodically during long operations

### Checkpoint Validity

```swift
func isValid(maxAge: TimeInterval = 3600) -> Bool {
    Date().timeIntervalSince(timestamp) < maxAge
}
```

Default to 1 hour validity. Older checkpoints are likely stale.

### Resume Flow

1. `loadInterruptedSessions()` scans checkpoint directory
2. Filter by `isValid(maxAge:)`
3. Show resume prompt to user
4. Restore messages and continue agent loop
5. On dismiss, delete checkpoint

## Background Execution

iOS gives approximately 30 seconds of background time. Use it wisely:

```swift
func prepareForBackground() {
    backgroundTaskId = UIApplication.shared
        .beginBackgroundTask(withName: "AgentProcessing") {
            handleBackgroundTimeExpired()
        }
}

func handleBackgroundTimeExpired() {
    for session in sessions where session.status == .running {
        session.status = .backgrounded
        Task { await saveSession(session) }
    }
}

func handleForeground() {
    for session in sessions where session.status == .backgrounded {
        Task { await resumeSession(session) }
    }
}
```

Use background time to:
- Complete current tool call if possible
- Checkpoint session state
- Transition gracefully to backgrounded state

## Storage Architecture

### iCloud-First with Local Fallback

```swift
func getStorageURL() -> URL {
    if let url = fileManager.url(forUbiquityContainerIdentifier: nil) {
        return url.appendingPathComponent("Documents")
    }
    return fileManager.urls(for: .documentDirectory, in: .userDomainMask)[0]
}
```

**Benefits:**
- Automatic sync across devices without infrastructure
- Backup without user action
- Graceful degradation when iCloud unavailable
- Users can access data outside the app

### Directory Structure

```
Documents/
├── AgentCheckpoints/        # Ephemeral session state
│   └── {sessionId}.checkpoint
├── AgentLogs/               # Debugging and history
│   └── {type}/{sessionId}.md
└── UserData/                # User's actual content
    └── {entity_type}/{entity_id}/
        ├── content.txt
        ├── metadata.json
        └── agent_log.md
```

### Cloud File States

Files may exist in iCloud but not be downloaded locally:

```swift
await StorageService.shared
    .ensureDownloaded(folder: .userdata, filename: "content.txt")
```

Always ensure availability before reading.

## On-Device vs Cloud

| Component | On-Device | Cloud |
|-----------|-----------|-------|
| Orchestration | ✓ | |
| Tool execution (files, photos, HealthKit) | ✓ | |
| LLM calls | | ✓ (API) |
| Checkpoints | ✓ (local files) | Optional via iCloud |
| Long-running agents | Limited by iOS | Possible with server |

The app needs network for reasoning but can access data offline. Design tools to degrade gracefully when network is unavailable.

## Conflict Handling

If agents and users write to the same files:

### Strategies

| Strategy | Trade-off |
|----------|-----------|
| Last write wins | Simple; changes can be lost |
| Check before writing | Skip if modified since read |
| Separate spaces | Agent → drafts/, user promotes |
| Append-only logs | Additive, never overwrites |
| File locking | Prevents edits while open |

### Practical Guidance

- Logs and status files rarely conflict
- For user-edited content, keep agent output separate
- iCloud adds complexity by creating conflict copies

## Mobile-Specific Tools

Mobile platforms offer unique context:
- **HealthKit**: Steps, heart rate, sleep data
- **Photos**: Image library access
- **Location**: GPS and places
- **Calendars**: Events and reminders

### Dynamic Capability Discovery

Instead of static mapping:

```swift
// Two tools handle everything
list_available_types() → ["steps", "heart_rate", "sleep", ...]
read_data(type) → reads any discovered type
```

When a new metric is added to HealthKit, the agent discovers it automatically.

## Long-Running Agent Considerations

For agents that need more than iOS background time allows:

1. **Server-side orchestrator**: Runs for hours, mobile app is viewer
2. **Chunked work**: Break into segments that complete within limits
3. **Background refresh**: iOS allows periodic background fetch
4. **Push notifications**: Server alerts app when work completes
