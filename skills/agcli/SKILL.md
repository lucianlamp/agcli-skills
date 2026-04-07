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

- `agcli` installed (`npm install -g github:lucianlamp/agcli` — private repo, collaborator access required) or available in PATH
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

# Machine-readable JSONL event stream
agcli send "your prompt" -f /path/to/project --event-stream

# Event stream with panel HTML snapshots
agcli send "your prompt" -f /path/to/project --event-stream --panel-stream

# Auto-continue: automatically send continuation prompts (default: 3 retries)
agcli send "your prompt" -f /path/to/project --auto-continue
agcli send "your prompt" -f /path/to/project --auto-continue 5 --json
```

**Flag restrictions:**
- `--json` and `--event-stream` cannot be combined
- `--async` and `--event-stream` cannot be combined
- `--auto-continue` and `--async` cannot be combined
- `--panel-stream` requires `--event-stream`

**Idle-wait behavior**: `send` automatically waits for Antigravity to finish any ongoing generation before sending the prompt. No manual polling is needed.

**Note**: `agcli send` defaults to Planning mode with the latest Opus Thinking model.

### Chat Management

```bash
agcli new -f /path/to/project              # Start a new chat
agcli continue -f /path/to/project         # Resume the latest chat
agcli resume --list -f /path/to/project    # List chat history
agcli resume 2 -f /path/to/project         # Open Nth chat
agcli stop -f /path/to/project --json      # Stop active generation
```

**Note**: `new`, `continue`, and `resume` do NOT support `--json`. Check exit code (0 = success, 1 = error) and stderr for error detection.

### Instance Management

```bash
agcli status -f /path/to/project --json    # Check connection status
agcli open -f /path/to/project --json      # Launch (auto-starts if needed)
agcli instances --json                     # List all running instances
agcli close -f /path/to/project --json     # Close instance
```

### Model & Mode Configuration

```bash
agcli model -f /path/to/project                           # Get current model
agcli model "claude-sonnet-4-5-20250514" -f /path/to/project  # Set model
agcli model --list -f /path/to/project                     # List available models
agcli mode -f /path/to/project                             # Get current mode
agcli mode "Planning" -f /path/to/project                  # Set mode
```

**Note**: `model` and `mode` do NOT support `--json`. Use `agcli mode --list` to see actual available mode names (they vary by Antigravity version).

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

### Agent Bridge (JSONL Protocol)

```bash
agcli agent-bridge -f /path/to/project
```

Starts a stdin/stdout JSONL bridge for programmatic control. Send one JSON object per line:

```json
{"op": "send", "prompt": "Fix the login bug", "folder": "/project"}
{"op": "status", "folder": "/project"}
{"op": "stop", "folder": "/project"}
{"op": "exit"}
```

Supported ops: `new`, `send`, `status`, `instances`, `open`, `stop`, `send-all`, `clear-pending`, `sync-pending`, `close`, `jobs`, `mode`, `model`, `continue`, `resume`, `exit`.

### Setup Commands

```bash
agcli install-skill     # Install agcli skill to ~/.claude/commands/
agcli install-mcp       # Configure agcli MCP server for Gemini CLI
```

## JSON Output Format

All `--json` outputs follow `{ ok: boolean, ... }` convention. Errors always include `{ ok: false, error: "message" }`.

### `send --json` — Success

```json
{
  "ok": true,
  "status": "succeeded",
  "streamingMode": "disabled_for_json",
  "folder": "/path/to/project",
  "port": 9000,
  "prompt": "Add rate limiting",
  "autoLaunched": false,
  "waited": true,
  "queuedInAg": false,
  "response": "I've added rate limiting to...",
  "responseDetails": {
    "changeOverview": ["src/auth.ts modified"],
    "artifactCards": [],
    "footerActions": []
  },
  "chatStatus": {
    "isGenerating": false,
    "hasInput": false,
    "messageCount": 6,
    "pendingMessageCount": 0,
    "hasPendingMessages": false,
    "hasSendAllButton": false
  },
  "taskReport": null,
  "resultSummary": "Added rate limiting middleware",
  "filesRead": ["src/auth.ts", "src/middleware.ts"],
  "filesModified": ["src/auth.ts"],
  "artifactFiles": [],
  "actions": ["Edited src/auth.ts"],
  "errors": [],
  "changeOverview": ["src/auth.ts modified"],
  "artifactCards": [],
  "artifactRefs": [],
  "artifactBody": null,
  "artifactBodySource": null,
  "footerActions": [],
  "pendingMessageCount": 0,
  "hasSendAllButton": false
}
```

**Key fields for agents:**
- `ok` — `true` if response is usable, `false` if AG returned an error (rate limit, server error, etc.)
- `errorKind` — present only when `ok: false`. Values: `"rate_limit"`, `"server_error"`, `"generation_error"`
- `status` — `"succeeded"` or `"queued_in_ag"` (see below)
- `queuedInAg` — `true` means the prompt was queued, NOT executed yet
- `response` — the AI's response text (`undefined` when `queuedInAg`)
- `filesModified` — files changed by the AI
- `errors` — any errors encountered during execution
- `continuationCount` — present only when `--auto-continue` was used and at least one continuation was sent

### `send --json` — AG Error Detected

When Antigravity returns a rate-limit, server error, or generation error instead of a real response, `ok` is `false` and `errorKind` indicates the type:

```json
{
  "ok": false,
  "errorKind": "rate_limit",
  "status": "succeeded",
  "response": "Our servers are experiencing high traffic right now, please try again in a minute.",
  "filesModified": [],
  "errors": []
}
```

**How to handle `ok: false`:**
- `rate_limit` — wait and retry after a delay
- `server_error` — retry, or escalate if persistent
- `generation_error` — rephrase or simplify the prompt

### `send --json` — With Auto-Continue

When `--auto-continue` is used, the response is the concatenation of all segments, and `continuationCount` shows how many continuation prompts were sent:

```json
{
  "ok": true,
  "status": "succeeded",
  "response": "First part of the response...\nContinuation 1...\nContinuation 2...",
  "continuationCount": 2,
  "filesModified": ["src/auth.ts"]
}
```

If a continuation triggers an AG error, the partial concatenation is returned with `ok: false`:

```json
{
  "ok": false,
  "errorKind": "rate_limit",
  "status": "succeeded",
  "response": "First part...\nRate limit exceeded",
  "continuationCount": 2
}
```

### `send --json` — Queued in AG (IMPORTANT)

When Antigravity is already generating a response, the prompt gets queued instead of executed:

```json
{
  "ok": true,
  "status": "queued_in_ag",
  "queuedInAg": true,
  "response": undefined,
  "pendingMessageCount": 1,
  "hasSendAllButton": true,
  "filesModified": [],
  "errors": []
}
```

**How to handle `queued_in_ag`:**
1. Wait for the current generation to finish: `agcli status -f /path --json` (check `chat.isGenerating`)
2. Dispatch queued messages: `agcli send-all -f /path --json`
3. Or sync job state: `agcli sync-pending -f /path --json`

### `send --async --json`

```json
{
  "ok": true,
  "async": true,
  "streamingMode": "disabled_for_async",
  "jobId": "uuid-string",
  "status": "queued",
  "folder": "/path/to/project",
  "port": 9000,
  "timeoutMs": 1800000
}
```

### `status --json`

```json
{
  "ok": true,
  "connected": true,
  "targetId": "target-abc123",
  "targetTitle": "my-app — Antigravity",
  "folder": "/path/to/project",
  "port": 9000,
  "chat": {
    "isGenerating": false,
    "hasInput": false,
    "messageCount": 5,
    "pendingMessageCount": 0,
    "hasPendingMessages": false,
    "hasSendAllButton": false
  }
}
```

Not connected: `{ "ok": false, "connected": false, "error": "..." }`

### `stop --json`

```json
{
  "ok": true,
  "folder": "/path/to/project",
  "port": 9000,
  "stopped": true,
  "method": "cancel_button"
}
```

`method` values: `"cancel_button"`, `"not_generating"`.
`stopped: false` means the cancel button was not found.

### `open --json`

```json
{ "ok": true, "folder": "/path", "port": 9000, "alreadyRunning": true }
{ "ok": true, "folder": "/path", "port": 9000, "alreadyRunning": false }
```

### `close --json`

```json
{ "ok": true, "alreadyClosed": true, "folder": "/path", "port": 9000 }
{ "ok": true, "alreadyClosed": false, "folder": "/path", "port": 9000, "closed": true }
```

### `instances --json`

```json
{
  "ok": true,
  "instances": [
    { "id": "target-1", "port": 9000, "title": "Antigravity — ProjectA", "workspace": "Antigravity", "url": "...", "wsUrl": "...", "score": 150 }
  ]
}
```

### `jobs --json` (list)

```json
{ "ok": true, "jobs": [ { "id": "uuid", "status": "succeeded", "prompt": "...", "folder": "...", "createdAt": "...", ... } ] }
```

### `jobs <id> --json` (detail)

```json
{ "ok": true, "job": { "id": "uuid", "status": "succeeded", "response": "...", "filesModified": [...], ... } }
```

**AsyncJobStatus values:** `queued`, `running`, `queued_in_ag`, `dispatched_in_ag`, `orphaned_in_ag`, `cancelled`, `succeeded`, `failed`, `timed_out`

**Active statuses** (still running): `queued`, `running`, `queued_in_ag`, `dispatched_in_ag`
**Terminal statuses** (finished): `succeeded`, `failed`, `timed_out`, `cancelled`, `orphaned_in_ag`

### `jobs --prune --json`

```json
{ "ok": true, "pruned": 3, "dryRun": false, "jobs": [...] }
```

### `send-all --json`

```json
{
  "ok": true,
  "folder": "/path",
  "port": 9000,
  "method": "send_all_button",
  "acted": true,
  "beforePendingCount": 2,
  "afterPendingCount": 0,
  "affectedJobIds": ["uuid-1", "uuid-2"]
}
```

### `clear-pending --json`

Same structure as `send-all --json` but with `method: "discard"`.

### `sync-pending --json`

```json
{
  "ok": true,
  "folder": "/path",
  "port": 9000,
  "pendingPrompts": ["Fix the bug", "Add tests"],
  "pendingMessageCount": 2,
  "hasSendAllButton": true,
  "confirmedCount": 1,
  "orphanedCount": 0,
  "confirmedJobIds": ["uuid-1"],
  "orphanedJobIds": []
}
```

## Exit Codes (`send` command)

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | Connection failure (WebSocket/ECONNREFUSED) |
| 2 | Timeout |
| 3 | Send/prompt error |

Other commands use exit code 1 for all errors.

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
# → {"ok": true, "async": true, "jobId": "abc123", ...}

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

### Pattern 5: Long Response with Auto-Continue

For tasks that produce long responses (code generation, analysis), use `--auto-continue` to automatically send "続けて" when AG stops mid-response:

```bash
# Default: up to 3 continuations
agcli send "Generate a comprehensive test suite for src/auth.ts" \
  -f /target/project --auto-continue --json

# More continuations for very long output
agcli send "Analyze all security vulnerabilities in this project" \
  -f /target/project --auto-continue 8 --json
```

Timeout applies **per segment**, not to the total. Each continuation resets the timer.

### Pattern 6: Error Recovery

```bash
# Reconnect if disconnected
agcli status -f /target/project --json || agcli open -f /target/project

# Stop stuck generation
agcli stop -f /target/project --json

# Clear jammed pending queue
agcli sync-pending -f /target/project --json
agcli clear-pending -f /target/project --json  # if sync doesn't resolve it
```

### Pattern 7: Handle queued_in_ag

```bash
# Send returns queued_in_ag — AG was busy
result=$(agcli send "Fix bug" -f /project --json)
status=$(echo "$result" | jq -r '.status')

if [ "$status" = "queued_in_ag" ]; then
  # Wait for generation to finish, then dispatch
  while agcli status -f /project --json | jq -e '.chat.isGenerating'; do
    sleep 5
  done
  agcli send-all -f /project --json
fi
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
| `status: "queued_in_ag"` returned | `agcli send-all` after generation finishes |
| Need structured real-time events | `agcli send --event-stream` |
| Task likely to produce long output | `agcli send --auto-continue` |
| `ok: false` with `errorKind: "rate_limit"` | Wait and retry after a delay |
| `ok: false` with `errorKind: "server_error"` | Retry, escalate if persistent |

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
- `--timeout` applies **per segment** when using `--auto-continue` — each continuation resets the timer
- `send` automatically waits for AG to finish generating before sending (no manual idle-polling needed)
- Avoid concurrent `send` to the same folder (chat input contention)
- Async jobs persist in `~/.agcli/jobs/`; run `--prune` periodically
- `send` and `open` auto-launch Antigravity if not running (`--no-auto-launch` to disable)
- Use `-v` / `--verbose` for debug logging (sets log level to debug)
- `model` and `mode` commands output plain text only (no `--json` support)
- Always check `ok` in `send --json` output — `false` means AG returned an error (check `errorKind`)
- Always check the `status` field in `send --json` output — `"queued_in_ag"` means NOT executed
