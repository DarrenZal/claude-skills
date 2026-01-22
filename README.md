# Claude Code Configuration

Personal configuration, skills, and MCP servers for [Claude Code](https://docs.anthropic.com/en/docs/claude-code).

## Structure

```
claude/
├── skills/              # Custom slash commands
│   ├── meeting-notes.md
│   └── transcribe.md
├── mcp-servers/         # MCP server setup guides
│   ├── otter.md
│   ├── google-workspace.md
│   ├── obsidian-vault.md
│   ├── playwright.md
│   └── chrome-devtools.md
├── CLAUDE.md.example    # Global CLAUDE.md template
└── docs/                # Tips and guides
```

## Quick Start

### Install Skills

```bash
# Copy skills to Claude Code commands directory
cp skills/*.md ~/.claude/commands/

# Use in Claude Code
/meeting-notes
/transcribe
```

### Configure MCP Servers

See [mcp-servers/README.md](mcp-servers/README.md) for setup guides.

```bash
# Example: Add Otter.ai MCP
claude mcp add -s user otter \
  -e OTTER_EMAIL=you@email.com \
  -e OTTER_PASSWORD=your_password \
  -- python -m otter_mcp
```

### Global CLAUDE.md

```bash
# Copy the example to enable global instructions
cp CLAUDE.md.example ~/.claude/CLAUDE.md
```

## Skills

Custom slash commands that extend Claude Code.

| Skill | Description |
|-------|-------------|
| [`/meeting-notes`](skills/meeting-notes.md) | Create Obsidian meeting notes from Otter.ai or MacWhisper |
| [`/transcribe`](skills/transcribe.md) | Transcribe YouTube videos |

### Creating Skills

Skills are markdown files with YAML frontmatter:

```markdown
---
allowed-tools:
  - Bash
  - Read
  - Write
description: Short description shown in /help
argument-hint: <required-arg> [optional-arg]
---

# Skill Title

Instructions for Claude Code...
```

## MCP Servers

Model Context Protocol servers extend Claude with external tools.

| Server | Description |
|--------|-------------|
| [Otter.ai](mcp-servers/otter.md) | Meeting transcript search |
| [Google Workspace](mcp-servers/google-workspace.md) | Gmail, Drive, Calendar, Docs |
| [Obsidian Vault](mcp-servers/obsidian-vault.md) | Read/write Obsidian notes |
| [Playwright](mcp-servers/playwright.md) | Browser automation |
| [Chrome DevTools](mcp-servers/chrome-devtools.md) | Browser debugging |

## Configuration Reference

| File | Location | Purpose |
|------|----------|---------|
| `~/.claude.json` | User home | MCP servers, user settings |
| `~/.claude/CLAUDE.md` | User config | Global instructions for all projects |
| `~/.claude/commands/*.md` | User config | Custom slash command skills |
| `~/.claude/settings.json` | User config | Enabled plugins |
| `.mcp.json` | Project root | Project-scoped MCP servers (shared) |
| `.claude/settings.local.json` | Project | Local project settings (not shared) |
| `CLAUDE.md` | Project root | Project-specific instructions |

## Tips

### MCP Server Management

```bash
# Add server (user scope - all projects)
claude mcp add -s user <name> -- <command>

# Add with env vars
claude mcp add -s user <name> -e KEY=val -- <command>

# List servers
claude mcp list

# Remove server
claude mcp remove <name> -s user
```

### Useful Commands

```
/help          # Show available commands
/mcp           # Show MCP server status
/memory        # Manage project memory
/config        # View/edit configuration
```

## License

MIT
