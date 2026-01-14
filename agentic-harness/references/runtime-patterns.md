# Runtime Patterns for Agentic Applications

Design patterns for agentic applications at runtime—how the built application learns and improves from user interactions.

## Table of Contents

- [Decision Logging](#decision-logging)
- [Observe → Formalize Pattern](#observe--formalize-pattern)
- [Prompt-Native Development](#prompt-native-development)
- [Emergent Capability as Design Goal](#emergent-capability-as-design-goal)

---

## Decision Logging

Agentic applications should log human decisions to learn patterns and improve over time.

### decisions.json Structure

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

### What to Log

- **Context**: What situation triggered the decision point
- **Options**: What choices were presented
- **Choice**: What the user selected
- **Reasoning**: Why (if provided or inferable)
- **Outcome**: What happened as a result (filled in later)

### Using Decision Logs

Over time, patterns emerge:
- "User always prefers simpler solutions when given complexity/simplicity tradeoff"
- "User prioritizes shipping over polish"
- "User prefers REST over GraphQL"

Agents can use these patterns to:
1. Pre-select likely preferences
2. Explain recommendations based on past decisions
3. Reduce decision fatigue by asking fewer questions

---

## Observe → Formalize Pattern

Traditional development:
```
Imagine what users want → Build it → See if you're right
```

Agent-native development:
```
Build capable foundation → Observe what users ask agents to do → Formalize patterns that emerge
```

### Implementation

1. **Log user requests**: Record what users ask agents to accomplish
2. **Identify patterns**: Look for recurring request types
3. **Formalize gradually**: Turn common patterns into dedicated tools or prompts

### Example

```
Week 1-2: Users ask "reorganize my files by date"
Week 3-4: Users ask "clean up my downloads folder"
Week 5: Pattern identified - file organization is common

Formalization options:
- Add a file_organizer prompt template
- Create an organize_files tool with common presets
- Add file organization examples to agent context
```

### What NOT to Formalize

- One-off requests
- Requests that vary significantly each time
- Requests where agent already handles well with primitives

Formalize only when:
- Pattern appears 3+ times
- Current approach is inefficient or error-prone
- Formalization genuinely improves the experience

---

## Prompt-Native Development

New features should follow this lifecycle:

### 1. Start as Prompt
Define the outcome in natural language:
```
"When user asks to refactor code, analyze the code structure,
identify duplication and poor patterns, propose specific changes,
and implement them one file at a time with verification."
```

### 2. Iterate in Prose
Change behavior by editing the prompt, not code:
```
"When user asks to refactor code, FIRST ask what aspect they want
to focus on (performance, readability, maintainability). Then..."
```

### 3. Graduate to Code (Maybe)
Only when:
- Requirements have stabilized
- Speed or reliability demands it
- The prompt approach has clear limitations

### 4. Many Features Stay as Prompts
Not everything needs to become code. Prompts are:
- Faster to iterate
- Easier to customize per-user
- More flexible for edge cases

### Example Feature Lifecycle

| Stage | Implementation |
|-------|----------------|
| Week 1 | Prompt: "Help users write tests by..." |
| Week 2 | Refined prompt with examples |
| Week 3 | Prompt + reference file with test patterns |
| Week 4 | Still a prompt—working well, no need to codify |
| Month 3 | Graduated to tool because speed matters for CI integration |

---

## Emergent Capability as Design Goal

The test for whether an application is truly agent-native:

> "Describe an outcome within your app's domain that you didn't build a specific feature for. Can an agent figure out how to accomplish it using available tools? If yes—you've built something agent-native."

### Designing for Emergence

1. **Provide atomic tools**: Small, composable primitives
2. **Expose state**: Let agents read and understand system state
3. **Allow experimentation**: Don't block unexpected tool combinations
4. **Trust agent judgment**: Let agents decide HOW to accomplish goals

### Example: Unexpected Capability

**Tools provided:**
- `read_file`, `write_file`, `list_dir`
- `run_tests`
- `git_commit`

**Expected use:**
- Edit code, run tests, commit changes

**Emergent capability:**
- User: "Find all files that import the old API and update them"
- Agent: Uses `list_dir` + `read_file` to find imports, `write_file` to update, `run_tests` to verify, `git_commit` to checkpoint

You didn't build a "find and replace across files" feature—it emerged from primitives.

### Anti-Pattern: Blocking Emergence

```python
# Bad: Only allows pre-defined workflows
def execute_workflow(workflow_name: str):
    if workflow_name == "refactor":
        do_refactor_steps()
    elif workflow_name == "add_feature":
        do_feature_steps()
    else:
        raise ValueError("Unknown workflow")
```

```python
# Good: Exposes primitives, agent decides workflow
tools = [read_file, write_file, run_command, git_operations]
# Agent combines these however needed for the task
```
