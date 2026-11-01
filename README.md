# LaneBuddy for AI agents

[LaneBuddy](https://lanebuddy.app) blocks ads, trackers and known threats for every app on a computer, watches every program start and stop, and (Max) watches the home network. This repository is what an AI agent needs to read and configure it:

- **`skills/lanebuddy`** — the `/lanebuddy` skill for Claude Code, Codex, Cursor and the other agents the [skills CLI](https://github.com/vercel-labs/skills) knows: which tool answers which sentence, the confirm rule, and the tool table.
- **`docs/MANUAL.md`** — the manual: every page, every setting, every finding kind, and a chapter on agent access.
- **`docs/API.md`** — the JSON API behind the dashboard, which the tools read.

## Install

`lane-mcp`, the local MCP server, ships beside LaneBuddy and finds the running engine by itself. Connect an agent, then add the skill:

```bash
claude mcp add --transport stdio lanebuddy -- lane-mcp        # Claude Code
npx skills add KurtStevenK/lanebuddy-ai --skill lanebuddy       # the skill, into every agent the CLI knows
```

Codex, in `~/.codex/config.toml`:

```toml
[mcp_servers.lanebuddy]
command = "lane-mcp"
```

Cursor, in the user or project MCP settings:

```json
{ "mcpServers": { "lanebuddy": { "command": "lane-mcp" } } }
```

Then `/lanebuddy how is my system doing`, `/lanebuddy which devices are on my LAN`, `/lanebuddy block the iPad`. A tool that blocks, stops, denies, deletes or sends something out shows a preview first and asks once.

The Settings page of LaneBuddy, under "AI agents", carries these lines with the real path filled in.

## Versions

`VERSION` is the LaneBuddy version this copy was taken from. The files are synced from LaneBuddy's own repository with each release that touches them.

## License

[MIT](LICENSE).
