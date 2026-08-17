# pool

pool is [Poolside](https://poolside.ai)’s coding agent. It can run in several modes:

- In your terminal as a standalone interactive application
- As an [ACP](https://agentclientprotocol.com/) server with [a compatible editor](#run-as-an-acp-server-pool-acp)
- As an ACP client connected to [an ACP server](#run-as-an-acp-client-pool---agent-server)
- Non-interactively with `pool exec`.

Unless noted otherwise, this README describes the default agent that ships with `pool`. When you connect to another ACP server with `--agent-server`, available modes and agent-side features depend on that server.

| Ghostty | Terminal.app |
| --- | --- |
| <img src="https://github.com/user-attachments/assets/7bff40ed-f347-4fcb-924f-85f4fd08ed82" /> | <img src="https://github.com/user-attachments/assets/2e91f838-501b-4449-b8d9-5779137cece0" /> |

## Contents

- [Install](#install)
- [Quick start](#quick-start)
- [Approval modes](#approval-modes)
- [Agent modes](#agent-modes)
- [Hooks](#hooks)
- [Subagents](#subagents)
- [Spec support](#spec-support)
- [Run as an ACP server (`pool acp`)](#run-as-an-acp-server-pool-acp)
- [Run as an ACP client (`pool --agent-server`)](#run-as-an-acp-client-pool---agent-server)
- [Run non-interactively (`pool exec`)](#run-non-interactively-pool-exec)
- [OpenRouter](#openrouter)
- [Ollama](#ollama)
- [OpenAI-compatible API](#openai-compatible-api)
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

To update, exit any active session and run `pool update`. `pool` also prompts you at startup when a newer version is available.

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
- Mid-turn steering when the connected agent server supports it
- Session management with `/resume` and `/delete`, and `/rename` when the connected agent server supports it

Enter `?` or `/help` during a session to see all available commands and shortcuts.

Enter another prompt while the agent is working and press `Enter` to steer the running turn. Press `Ctrl+Enter` to queue the prompt for the next turn when your terminal supports key disambiguation. Shell input and slash commands wait until the current turn finishes. If another ACP server does not support steering, the prompt waits for the next turn.

## Approval modes

`pool` asks for approval by default before tool actions that are not already allowed. Switch to Accept edits, Auto, or Allow all to change which actions require confirmation. Other ACP servers can provide different approval modes.

| Approval mode | ID | What it does |
|---|---|---|
| Always ask | `default` | Prompts for tool actions that are not already allowed |
| Accept edits | `accept-edits` | Auto-approves workspace file reads and writes |
| Auto | `auto` | Classifies remaining permission requests by risk |
| Allow all | `always-allow` | Approves tool calls automatically |

Press `Shift+Tab` to cycle through approval modes, or use `/mode <approval-mode>` to switch directly.

Auto mode uses Accept edits as a baseline. It runs low-risk actions immediately, runs medium-risk actions with a notice, and opens the approval dialog for high-risk actions. Configure a classifier model in your personal `~/.config/poolside/settings.yaml` file:

```yaml
pool:
  auto_mode_classifier: <model-id>
```

To enable or override Auto mode for one invocation, set `POOL_AUTO_MODE_CLASSIFIER_MODEL` when you start `pool`. An explicitly empty value disables Auto mode for that invocation.

See [Permissions](https://docs.poolside.ai/permissions) for classifier inputs, failure behavior, and safety constraints.

## Agent modes

Build and Plan control how pool works without changing approval behavior. In Plan mode, pool can inspect your codebase and prepare a plan without modifying source files.

After starting pool, use `/plan` or `/agent-mode plan` to enter Plan mode. Use `/agent-mode build` to return to Build mode. There is no Plan-mode startup flag. Returning from Plan to Build always requires your review, including when the approval mode is Allow all.

## Hooks

Use hooks to run shell commands at six agent lifecycle events: `PreToolUse`, `PostToolUse`, `UserPromptSubmit`, `Stop`, `PreCompact`, and `SessionStart`. Hooks can inspect or rewrite tool calls and prompts, block matching actions, inject context, and ask the agent to continue at the end of a turn.

Configure hooks under the top-level `hooks` key in `settings.yaml`.

Hooks run automatically without an approval prompt and fail open if they fail, time out, or return output `pool` cannot parse. They are not a security boundary. Use permissions and sandboxes for enforced controls. See [Hooks](https://docs.poolside.ai/hooks) for configuration, protocol details, examples, and security guidance.

## Subagents

The `general` subagent requires no configuration and lets the main agent delegate focused work with separate context. Ask the main agent to delegate a focused task to it. You can also configure named subagents that use custom instructions, another in-process Poolside agent, or a command-based ACP server.

Subagents share the workspace, so parallel changes to the same files can conflict. Run `/usage` to see parent, per-subagent, and total usage. See [Subagents](https://docs.poolside.ai/subagents) for configuration, permissions, and usage accounting.

## Spec support

pool implements and integrates with open agent specs:

- **[AGENTS.md](https://agents.md)**: pool automatically reads relevant `AGENTS.md` files from your project for project context and instructions.
- **[Skills](https://agentskills.io)**: load skills to extend pool with reusable workflows.
- **[MCP](https://modelcontextprotocol.io)**: connect tools and data sources via [MCP servers](#mcp-servers).
- **[ACP](https://agentclientprotocol.com)**: runs as an [ACP server](#run-as-an-acp-server-pool-acp) inside editors and as an [ACP client](#run-as-an-acp-client-pool---agent-server) driving other agents.

## Run as an ACP server (`pool acp`)

`pool acp` is an [Agent Client Protocol](https://agentclientprotocol.com) server. Use it from any ACP-compatible client, like [Zed](https://zed.dev/acp), [JetBrains](https://www.jetbrains.com/acp/), [Xcode](https://developer.apple.com/documentation/Xcode/setting-up-coding-intelligence), and [others](https://agentclientprotocol.com/get-started/clients).

Install it from the [ACP registry](https://agentclientprotocol.com/get-started/registry).

### Other editors

Point the editor's ACP configuration at `pool acp`:

```json
{
  "command": "pool",
  "args": ["acp"]
}
```

To pass flags to the ACP server, add them to the args array, for example `["acp", "--sandbox", "required"]`.

### ACP features

- Session persistence: `session/list` and `session/load`
- Mid-turn steering via the `poolside/session_steer` capability
- Session config options: approval mode, agent mode, model, and thought level when supported. These can be persisted in `settings.yaml`
  and are sent on startup using `session/set_config_option`
- Slash commands advertised to the client

## Run as an ACP client (`pool --agent-server`)

By default `pool` connects to Poolside's built-in agent through `pool acp`. You can also connect it to any other [ACP server](https://agentclientprotocol.com/get-started/agents):

```bash
# Claude Agent
npm install -g @agentclientprotocol/claude-agent-acp
pool --agent-server claude-agent-acp

# Codex
npm install -g @agentclientprotocol/codex-acp
pool --agent-server codex-acp
```

To set other ACP agent as a default, change `pool.default_agent_server` key:

```yaml
pool:
  default_agent_server: claude
agent_servers:
  claude:
    command: claude-agent-acp
```

Flags after `--` are forwarded to the ACP server `pool` is running. For example:

```bash
pool -- --sandbox required
```

You can also use configure remote ACP agents that work through [streamable HTTP transport](https://agentclientprotocol.com/rfds/streamable-http-websocket-transport):

```yaml
# settings.yaml
agent_servers:
  remote:
    url: https://my-vm.exe.xyz/acp
    headers:
      X-Secret-Key: my-secret
```

And connect with `pool -s remote`.

## Run non-interactively (`pool exec`)

Run `pool exec` to send a single prompt and exit when the task is complete. Use this for scripts, CI pipelines, and one-off tasks.

```bash
# Inline prompt
pool exec -p "scan cmd/cli code for vulnerabilities" -o json --unsafe-auto-allow

# Prompt from a file
pool exec -f prompt.txt -o json
```

Running `pool exec` non-interactively does not select standalone mode automatically. In CI or another headless environment without credentials saved by `pool login`, set the API key and base URL for your provider:

```bash
POOLSIDE_API_KEY="<api-key>" \
  POOLSIDE_STANDALONE_BASE_URL="<provider-base-url>" \
  pool exec -p "<prompt>" -o json --unsafe-auto-allow
```

For Poolside Platform, use `https://inference.poolside.ai` as the base URL.

To choose a model instead of using the default, also set `POOLSIDE_STANDALONE_MODEL` to its model ID.

## OpenRouter

[OpenRouter](https://openrouter.ai) is supported natively in `pool`.

Run `pool login` and select `Use your OpenRouter account`.

## Ollama

[Ollama](https://ollama.com) supports `pool` natively:

```bash
# Launch model selector
ollama launch pool

# Launch directly with a model
ollama launch pool --model laguna-xs.2
```

## OpenAI-compatible API

You can run `pool` against any OpenAI-compatible API. For example, for using local models with [`llama.cpp`](https://llama.app) or locally running [vLLM](https://vllm.ai).

```bash
POOLSIDE_STANDALONE_BASE_URL="http://127.0.0.1:8080" POOLSIDE_API_KEY="EMPTY" pool
```

Optional environment variables you can pass:

- `POOLSIDE_STANDALONE_MODEL` to set the model
- `POOLSIDE_STANDALONE_CONTEXT_LENGTH` to override, in tokens, the context length `pool` uses to calculate automatic compaction thresholds

```bash
POOLSIDE_STANDALONE_BASE_URL="http://127.0.0.1:8080" \
POOLSIDE_API_KEY="EMPTY" \
POOLSIDE_STANDALONE_MODEL="ggml-org/gemma-3-1b-it-GGUF" \
POOLSIDE_STANDALONE_CONTEXT_LENGTH=200000 \
pool
```

Use a positive `POOLSIDE_STANDALONE_CONTEXT_LENGTH` value no greater than the model server's context window. The override does not change that window and applies only when the selected model appears in the provider's `/v1/models` response.

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

By default, Poolside stores configuration files in `~/.config/poolside`. This includes `settings.yaml` (CLI settings), `credentials.json` (API token).

For automation environments without credentials saved by `pool login`, set `POOLSIDE_API_KEY` and the provider endpoint. See [Run non-interactively (`pool exec`)](#run-non-interactively-pool-exec).

### settings.yaml reference

Most settings can be set globally (`~/.config/poolside/settings.yaml`) or
per-project (`.poolside/settings.yaml`, or gitignored
`.poolside/settings.local.yaml`). Merge behavior varies by setting: more
specific values usually override less specific ones, but some combine across
files. See the
[settings.yaml reference](https://docs.poolside.ai/settings-file-reference) for
details. The `pool` keys below are an exception. `pool` reads them only from
`~/.config/poolside/settings.yaml`.

Besides [agent servers](#run-as-an-acp-client-pool---agent-server),
[MCP servers](#mcp-servers), [hooks](#hooks), [subagents](#subagents), and
[permissions](#permissions), `settings.yaml` supports:

**CLI settings**:

```yaml
pool:
  auto_mode_classifier: <model-id>   # classifier used by Auto approval mode
  default_agent_server: my-acp-agent # agent used when --agent-server is omitted
  worktree_prefix: alice-            # prefix for auto-generated --worktree names
```

**System prompt override**:

```yaml
prompts:
  system_message: |
    You are a security-focused code reviewer. Flag unsafe patterns
    and never suggest disabling TLS verification.
```

This fully replaces the built-in system prompt. To add instructions on top of the default prompt, use global [AGENTS.md](https://agents.md) file instead.

**Web search**:

```yaml
web_search:
  provider: exa             # "exa" or "parallel"
  api_key: your-api-key
  summarize_results: true   # summarize fetched pages with the session model
```

**Disabling a tool**:

```yaml
tools:
  web_fetch:
    disabled: true
```

## Permissions

Permission rules can be set at three scopes:

- `.poolside/settings.local.yaml` – local, per-project (gitignored)
- `.poolside/settings.yaml` – shared per-project (checked in)
- `~/.config/poolside/settings.yaml` – personal defaults across projects

Poolside combines tool and path `allow` and `deny` rules across all files. A matching `deny` overrides `allow`.

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
