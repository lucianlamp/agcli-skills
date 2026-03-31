---
name: agcli
description: Headless control of Antigravity IDE via CLI and CDP. Use when delegating tasks to Antigravity, sending prompts, managing chat sessions, controlling instances, running parallel AI agent tasks, or automating IDE workflows from the terminal.
allowed-tools: Bash(agcli:*)
---

# agcli — Antigravity Headless Automation Skill

Control Antigravity IDE headlessly via the `agcli` CLI. Send prompts, manage chat sessions, control instances, and orchestrate parallel AI agent workflows — all from the command line.

## When to Use This Skill

- Delegate a task to Antigravity (send a prompt, wait for response)
- Manage Antigravity chat sessions (new, continue, resume, stop)
- Check or control Antigravity instance state (status, open, close)
- Run parallel AI agent tasks across multiple projects
- Automate IDE workflows from scripts or other agents

## Prerequisites

- `agcli` installed (`npm install -g agcli` or available in PATH)
- Antigravity with CDP enabled (default port: 9000)
- Use `--json` flag for machine-readable output in automation

## Command Reference

### Send Prompts (Most Common)

```bash
# Basic: send prompt, wait for streaming response
agcli send "your prompt" -f /path/to/project

# JSON output (for parsing)
agcli send "your prompt" -f /path/to/project --json

# Async: returns job ID immediately, runs in background
agcli send "your prompt" -f /path/to/project --async --json

# Disable streaming
agcli send "your prompt" -f /path/to/project --no-stream --json

# Custom timeout (ms)
agcli send "your prompt" -f /path/to/project --timeout 600000

# Show thinking tokens (debug)
agcli send "your prompt" -f /path/to/project --show-thinking
```

**Note**: `agcli send` defaults to Planning mode with the latest Opus Thinking model.

### Chat Management

```bash
agcli new -f /path/to/project              # Start a new chat
agcli continue -f /path/to/project         # Resume the latest chat
agcli resume --list -f /path/to/project    # List chat history
agcli resume 2 -f /path/to/project         # Open Nth chat
agcli stop -f /path/to/project --json      # Stop active generation
```

### Instance Management

```bash
agcli status -f /path/to/project --json    # Check connection status
agcli open -f /path/to/project             # Launch (auto-starts if needed)
agcli instances --json                     # List all running instances
agcli close -f /path/to/project --json     # Close instance
```

### Model & Mode Configuration

```bash
agcli model -f /path/to/project                           # Get current model
agcli model "claude-sonnet-4-5-20250514" -f /path/to/project  # Set model
agcli model --list -f /path/to/project                     # List available models
agcli mode -f /path/to/project                             # Get current mode
agcli mode "Ask" -f /path/to/project                       # Set mode (Agent, Ask, Edit)
```

### Job Management (Async)

```bash
agcli jobs --active --json              # List active jobs
agcli jobs <job-id> --json              # Job details
agcli jobs <job-id> --wait --json       # Wait for completion
agcli jobs --prune --json               # Clean up old jobs
```

### Pending Message Operations

```bash
agcli send-all -f /path/to/project --json       # Send all pending messages
agcli clear-pending -f /path/to/project --json   # Discard pending messages
agcli sync-pending -f /path/to/project --json    # Sync job state with AG queue
```

## Workflow Patterns

### Pattern 1: Single Task Delegation

```bash
agcli status -f /target/project --json
agcli new -f /target/project
agcli send "Add rate limiting to the login function in src/auth.ts" \
  -f /target/project --json
```

### Pattern 2: Async Task + Parallel Work

```bash
# Fire and forget
agcli send "Increase test coverage to 80%+" \
  -f /target/project --async --json
# → {"jobId": "abc123", ...}

# ... do other work ...

# Check result later
agcli jobs abc123 --wait --json
```

### Pattern 3: Continuous Dialog

```bash
agcli new -f /target/project
agcli send "Analyze this project's architecture" -f /target/project --json
agcli send "Explain the auth flow in detail" -f /target/project --json
```

### Pattern 4: Multi-Project Parallel

```bash
agcli send "Fix lint errors" -f /project-a --async --json
agcli send "Update type definitions" -f /project-b --async --json
agcli jobs --active --json
```

### Pattern 5: Error Recovery

```bash
# Reconnect if disconnected
agcli status -f /target/project --json || agcli open -f /target/project

# Stop stuck generation
agcli stop -f /target/project --json

# Clear jammed pending queue
agcli sync-pending -f /target/project --json
agcli clear-pending -f /target/project --json  # if sync doesn't resolve it
```

## JSON Output Format

All `--json` outputs follow `{ ok: boolean, ... }` convention.

**send success:**
```json
{
  "ok": true,
  "response": "Response text...",
  "responseDetails": {
    "changeOverview": ["file.ts modified"],
    "artifactCards": [],
    "footerActions": []
  },
  "filesModified": ["src/auth.ts"],
  "pendingMessageCount": 0
}
```

**async send:**
```json
{
  "ok": true,
  "async": true,
  "jobId": "uuid",
  "status": "queued",
  "folder": "/path/to/project"
}
```

**status:**
```json
{
  "ok": true,
  "connected": true,
  "targetTitle": "my-app — Antigravity",
  "chat": {
    "isGenerating": false,
    "messageCount": 5,
    "pendingMessageCount": 0
  }
}
```

## Decision Guidelines

| Situation | Recommended Action |
|-----------|-------------------|
| Task you can implement directly | Do it yourself, don't use agcli |
| Task in a different project | `agcli send -f /other/project` |
| Long-running investigation | `agcli send --async` |
| Unknown Antigravity state | `agcli status --json` first |
| Continue previous conversation | `agcli continue` (new chat: `agcli new`) |
| Response not coming back | `agcli stop` then `agcli status` |

## Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `ANTIGRAVITY_CDP_PORT` | CDP port | `9000` |
| `ANTIGRAVITY_USER_DATA_DIR` | Chromium profile dir | (none) |
| `ANTIGRAVITY_PATH` | Executable path | (auto-detect) |
| `AGCLI_DOM_DRIVER` | DOM driver | `playwright` |
| `AGCLI_LOG_LEVEL` | Log level | `warn` |

## Important Notes

- Always specify `-f` / `--folder` to target the correct instance
- Default send timeout is 30 minutes (1800000ms); use `--timeout` for longer tasks
- Avoid concurrent `send` to the same folder (chat input contention)
- Async jobs persist in `~/.agcli/jobs/`; run `--prune` periodically
- `send` and `open` auto-launch Antigravity if not running (`--no-auto-launch` to disable)
