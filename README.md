# pool

pool is [Poolside](https://poolside.ai)’s coding agent. It can run in several modes:

- In your terminal as a standalone interactive application
- As an [ACP](https://agentclientprotocol.com/) server with [a compatible editor](#run-as-an-acp-server-pool-acp)
- As an ACP client connected to [an ACP server](#run-as-an-acp-client-pool---agent-server)
- Non-interactively with `pool exec`.

| Ghostty | Terminal.app |
| --- | --- |
| <img src="https://github.com/user-attachments/assets/7c5befea-2e12-4aa1-809e-e9f25b057362" /> | <img src="https://github.com/user-attachments/assets/d2088fa4-bf1b-438b-aec5-57dbb6716c0b" /> |

## Contents

- [Install](#install)
- [Quick start](#quick-start)
- [Modes](#modes)
- [Spec support](#spec-support)
- [Run as an ACP server (`pool acp`)](#run-as-an-acp-server-pool-acp)
- [Run as an ACP client (`pool --agent-server`)](#run-as-an-acp-client-pool---agent-server)
- [Run non-interactively (`pool exec`)](#run-non-interactively-pool-exec)
- [MCP servers](#mcp-servers)
- [Configuration](#configuration)
- [Permissions](#permissions)
- [Feedback and bugs](#feedback-and-bugs)

## Install

Linux and macOS:

```bash
curl -fsSL https://downloads.poolside.ai/pool/install.sh | sh
```

Windows (preview):
```pwsh
irm https://downloads.poolside.ai/pool/install.ps1 | iex
```

## Quick start

Run `pool` in any project:

```bash
cd your-project
pool
```

Run `pool -h` to see all available options.

### Interactive features

- Slash commands with `/`
- Fuzzy search over files and directories with `@`
- Shell mode with `!`
- Rewind to previous messages with double `esc`

Enter `?` or `/help` during a session to see all available commands and shortcuts.

## Modes

By default, pool asks for approval before each tool call. Switch to Accept edits or Allow all when you want it to take actions without prompting.

| Mode          | ID             | What it does                                                       |
| ------------- | -------------- | ------------------------------------------------------------------ |
| Always ask    | `default`      | Prompts for approval on first use of each tool type                |
| Accept edits  | `accept-edits` | Auto-approves workspace file reads and writes                      |
| Allow all     | `allow-all`    | Approves tool calls automatically                                  |
| Plan          | `plan`         | Plans changes without modifying your codebase                      |

Press `Shift+Tab` to cycle through modes, or use `/mode <id>` to switch directly.

## Spec support

pool implements and integrates with open agent specs:

- **[AGENTS.md](https://agents.md)**: pool automatically reads relevant `AGENTS.md` files from your project for project context and instructions.
- **[Skills](https://agentskills.io)**: load skills to extend pool with reusable workflows.
- **[MCP](https://modelcontextprotocol.io)**: connect tools and data sources via [MCP servers](#mcp-servers).
- **[ACP](https://agentclientprotocol.com)**: runs as an [ACP server](#run-as-an-acp-server-pool-acp) inside editors and as an [ACP client](#run-as-an-acp-client-pool---agent-server) driving other agents.

## Run as an ACP server (`pool acp`)

`pool acp` is an [Agent Client Protocol](https://agentclientprotocol.com) server. Use it from any ACP-compatible client.

### Zed

```bash
pool acp setup --editor zed
```

This writes Poolside agent configuration to `~/.config/zed/settings.json`. Or add it manually:

```json
{
  "agent_servers": {
    "Poolside": {
      "command": "pool",
      "args": ["acp"],
      "type": "custom"
    }
  }
}
```

### JetBrains

```bash
pool acp setup --editor jetbrains
```

This writes configuration to `~/.jetbrains/acp.json`. Or add it manually:

```json
{
  "agent_servers": {
    "Poolside": {
      "command": "pool",
      "args": ["acp"]
    }
  }
}
```

### Other editors

Point the editor's ACP configuration at `pool acp`:

```json
{
  "command": "pool",
  "args": ["acp"]
}
```

To pass flags to the ACP server, add them to the args array, for example `["acp", "--reasoning", "high"]`.

### ACP features

- Session persistence: `session/list` and `session/load`
- Session config options: mode and model. These can be persisted in `pool.json`
  and are sent on startup using `session/set_config_option`
- Slash commands advertised to the client

## Run as an ACP client (`pool --agent-server`)

By default `pool` connects to Poolside's built-in agent through `pool acp`. You can also connect it to any other [ACP server](https://agentclientprotocol.com/get-started/agents):

```bash
# Claude Agent
npm install -g @agentclientprotocol/claude-agent-acp
pool --agent-server claude-agent-acp

# Codex
npm install -g @zed-industries/codex-acp
pool --agent-server codex-acp

# Gemini
npm install -g @google/gemini-cli
pool --agent-server "gemini --acp"
```

To set a non-Poolside server as the default, edit `~/.config/poolside/pool.json`:

```json
{
  "agent_servers": {
    "default": {
      "command": "claude-agent-acp"
    }
  }
}
```

Flags after `--` are forwarded to the ACP server `pool` is running. For example:

```bash
pool -- --reasoning high
```

## Run non-interactively (`pool exec`)

Run `pool exec` to send a single prompt and exit when the task is complete. Use this for scripts, CI pipelines, and one-off tasks.

```bash
# Inline prompt
pool exec -p "scan cmd/cli code for vulnerabilities" -o json --unsafe-auto-allow

# Prompt from a file
pool exec -f prompt.txt -o json
```

## MCP servers

`pool` can connect to [Model Context Protocol](https://modelcontextprotocol.io) servers to expose extra tools to the agent. Manage them with `pool mcp`:

```bash
# Command-based (stdio)
pool mcp add filesystem -- node filesystem-server.js

# Remote HTTP server
pool mcp add --transport http notion https://mcp.notion.com/mcp

# Remote SSE server
pool mcp add --transport sse linear https://mcp.linear.app/sse

# Pass environment variables or HTTP headers
pool mcp add --env API_KEY=your-key myserver -- npx -y myserver-mcp
pool mcp add --transport http --header "Authorization: Bearer $TOKEN" svc https://example.com/mcp

# Inspect and remove
pool mcp list
pool mcp get <name>
pool mcp remove <name>
```

Servers are stored under `mcp_servers` in `~/.config/poolside/settings.yaml`. You can also edit that file directly, or scope servers per-project by adding them to `.poolside/settings.yaml`.

## Configuration

Run `pool config` to print the log, trajectory, and configuration directories, as well as the credentials file path. Run `pool config settings` to open `settings.yaml` in your editor.

By default, Poolside stores configuration files in `~/.config/poolside`. This includes `settings.yaml` (CLI settings), `credentials.json` (API token), and `pool.json` (agent server defaults).

For automation environments, set `POOLSIDE_API_KEY` instead of using stored credentials. `pool` checks it before reading from configuration files.

## Permissions

Permission rules can be set at three scopes:

- `.poolside/settings.local.yaml` – local, per-project (gitignored)
- `.poolside/settings.yaml` – shared per-project (checked in)
- `~/.config/poolside/settings.yaml` – personal defaults across projects

When the same setting appears in multiple files, the most specific file wins.

### Tool permissions

```yaml
tools:
  shell:
    allow:
      - "git log *"
      - "rg *"
    deny:
      - "rm *"
      - "git push *"
```

How tool rules work:

- Tool rules support only `*` wildcards (`**` is not supported)
- The rule string must match the tool call shown in the approval prompt
- Subshells and composite shell commands always require manual approval
- Shell commands that use control operators such as `|` are not auto-approved

### Path permissions

```yaml
paths:
  allow:
    - path: ~/Documents/**
    - path: ~/workspace/docs/**
      write: true
  deny:
    - path: ~/.ssh/**
    - path: ~/.env
```

How path rules work:

- Paths are read-only by default; `write: true` allows edits, deletes, moves, renames
- `deny` overrides `allow`
- Path patterns support `*` and `**`
- Use forward slashes for all paths, including Windows paths
- In `.poolside/settings.local.yaml`, paths must be relative to the project
- In `~/.config/poolside/settings.yaml`, paths must be absolute or start with `~`

## Feedback and bugs

Run `/feedback` in an interactive session to send feedback or report a bug. To attach an earlier session, resume it first with `pool -r` and then run `/feedback`.

## License

See [LICENSE.md](https://github.com/poolsideai/pool/blob/main/LICENSE.md).
