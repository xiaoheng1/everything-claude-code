# Hooks System

> Official documentation: https://docs.anthropic.com/en/docs/claude-code/hooks

## Hook Types

| Type | Trigger | Use Case |
|------|---------|----------|
| **PreToolUse** | Before tool execution | Validation, interception, parameter modification |
| **PostToolUse** | After tool execution | Formatting, checking results, logging |
| **Stop** | When session ends | Cleanup, save state, final verification |
| **Notification** | When Claude sends notification | Custom notification handling |

## Configuration Location

| Scope | Location |
|-------|----------|
| Project | `.claude/settings.json` |
| User | `~/.claude/settings.json` |
| Enterprise | `/etc/claude-code/settings.json` |

## Configuration Structure

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": ["path/to/pre-bash-hook.sh"],
        "timeout": 60000
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Write",
        "hooks": ["path/to/post-write-hook.sh"]
      }
    ]
  }
}
```

| Field | Description |
|-------|-------------|
| `matcher` | Regex pattern to match tool name (e.g., `"Bash"`, `"Write"`, `"Read"`) |
| `hooks` | Array of commands/scripts to run |
| `timeout` | Optional timeout in milliseconds (default: 60000) |

## Environment Variables

Available in hook scripts:

| Variable | Description | Available In |
|----------|-------------|--------------|
| `CLAUDE_TOOL_NAME` | Tool name (e.g., `Write`, `Bash`, `Read`) | PreToolUse, PostToolUse |
| `CLAUDE_TOOL_INPUT` | JSON-encoded tool input parameters | PreToolUse, PostToolUse |
| `CLAUDE_TOOL_OUTPUT` | JSON-encoded tool output | PostToolUse |

## Hook Execution Flow

```
Tool invocation request
    ↓
PreToolUse hook(s)
    ├─ Match tool name?
    ├─ Run hook script
    ├─ Exit 0 → Continue to tool execution
    └─ Exit non-zero → Block tool, return error
    ↓
Tool execution (Write, Bash, Read, etc.)
    ↓
PostToolUse hook(s)
    ├─ Match tool name?
    ├─ Run hook script
    └─ Log/process result
    ↓
Return result to Claude
```

## Example Hooks

### 1. Format JavaScript files after Write

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write",
        "hooks": ["prettier --write $CLAUDE_TOOL_INPUT"]
      }
    ]
  }
}
```

### 2. Block dangerous Bash commands

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          "if [[ \"$CLAUDE_TOOL_INPUT\" =~ rm.*-rf ]]; then echo 'Dangerous command blocked' && exit 1; fi"
        ]
      }
    ]
  }
}
```

### 3. Session end cleanup

```json
{
  "hooks": {
    "Stop": [
      {
        "matcher": "",
        "hooks": ["echo 'Session ended at $(date)' >> ~/.claude/session-log.txt"]
      }
    ]
  }
}
```

## Auto-Accept Permissions

Use with caution:
- Enable for trusted, well-defined plans
- Disable for exploratory work
- NEVER use `dangerously-skip-permissions` flag
- Configure `allowedTools` in `settings.json` instead

## TodoWrite Best Practices

Use TodoWrite tool to:
- Track progress on multi-step tasks
- Verify understanding of instructions
- Enable real-time steering
- Show granular implementation steps

Todo list reveals:
- Out of order steps
- Missing items
- Extra unnecessary items
- Wrong granularity
- Misinterpreted requirements
