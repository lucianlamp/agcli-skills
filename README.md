# agcli-skills

Agent skills for [agcli](https://github.com/lucianlamp/agcli) — headless control of Antigravity IDE via CLI and CDP.

## Install

```bash
npx skills add lucianlamp/agcli-skills
```

Or for a specific agent:

```bash
npx skills add lucianlamp/agcli-skills -a claude
```

## What's included

| Skill | Description |
|-------|-------------|
| `agcli` | Send prompts, manage chat sessions, control instances, and orchestrate parallel AI agent workflows from the terminal |

### Key capabilities

- **Send prompts** with streaming, JSON, or JSONL event-stream output
- **Auto-continue** (`--auto-continue`) — automatically send continuation prompts when AG stops mid-response
- **Idle-wait** — `send` automatically waits for AG to finish generating before sending
- **Error detection** — `ok`/`errorKind` fields for structured error handling (rate limits, server errors)
- **Async jobs** — fire-and-forget with background job tracking
- **Multi-project** — orchestrate parallel tasks across different folders

## Prerequisites

- `agcli` installed and available in PATH (`npm install -g @lucianlamp/agcli`)
- Antigravity IDE with CDP enabled (default port: 9000)

## License

MIT
