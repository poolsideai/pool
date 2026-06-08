# Changelog

## [1.0.5] - 2026-06-08

- Notifications for turn end, permissions and elicitations, only emitted when `pool` is not focused
- Support for [cmux](https://cmux.com): notifications and status icons
- Rename session with the `/rename` command, find session again with `pool -r`
- See connected MCP servers with `/mcp` command
- Launch interactive ACP agent server picker with `pool -s`
- Added a built-in introspection skill: ask `pool` about itself
- Better text selection
- Moved `pool acp --reasoning` to session config options
- Fixed text pasting on Terminal.app
- Removed 30 second timeout for MCP tools

## [1.0.4] - 2026-05-27

- Fix scrolling for slash commands

## [1.0.3] - 2026-05-27

- Show git branch in the status line
- Show "Accept edits for this session" option in the permission dialog
- Support image pasting with Ctrl+V
- Expose skills as ACP slash commands
- Add `/compact` command, it can optionally take a guidance argument, e.g. `/compact preserve tool call errors`
- Better display for ANSI sequences in shell tool output
- Faster initial rendering for resumed sessions
- In `pool --resume`, see sessions from all directories with <Tab>

## [1.0.2] - 2026-05-14

- Show common shortcuts on startup
- New status line UI, show context usage
- Allow to use `pool` without login with other ACP servers or Ollama
- Markdown rendering fixes
- Show edit and write previews before asking for permissions
- `pool acp`: More detailed `/usage` slash command output
- `pool acp`: Fix `session/load` race condition
- `pool acp`: Implement ACP `session/close`

## [1.0.1] - 2026-05-04

- Allow more characters in paths stored in settings.yaml
- Show basedir in the terminal title
- Better installation experience on Windows
- Fix for MCP servers that require OAuth
- Add --prompt-queue (-q) flag to enqueue prompts on start
- Discover skills from .agent/skills

## [1.0.0] - 2026-04-28

- Initial public release
