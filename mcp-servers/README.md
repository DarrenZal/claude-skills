# MCP Server Configurations

Model Context Protocol (MCP) servers extend Claude Code with external tools and data sources.

## Configuration Location

MCP servers are configured in `~/.claude.json` under the `mcpServers` key. Use the `claude mcp add` command to add servers properly.

## Adding MCP Servers

```bash
# Basic syntax
claude mcp add -s user <name> -- <command> [args...]

# With environment variables
claude mcp add -s user <name> -e KEY=value -e KEY2=value2 -- <command> [args...]
```

## Available Configurations

| Server | Description | Setup |
|--------|-------------|-------|
| [Otter.ai](./otter.md) | Meeting transcript search & retrieval | API credentials |
| [Google Workspace](./google-workspace.md) | Gmail, Drive, Calendar, Docs | OAuth setup |
| [Obsidian Vault](./obsidian-vault.md) | Read/write Obsidian notes | Local path |
| [Playwright](./playwright.md) | Browser automation | npm package |
| [Chrome DevTools](./chrome-devtools.md) | Browser debugging | npm package |

## Scopes

- **user** (`-s user`): Available in all projects for this user
- **project** (`-s project`): Shared with team via `.mcp.json`
- **local** (`-s local`): Private to you in this project

## Removing Servers

```bash
claude mcp remove <name> -s user
```

## Listing Servers

```bash
claude mcp list
# or in Claude Code:
/mcp
```
