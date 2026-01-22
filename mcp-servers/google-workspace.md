# Google Workspace MCP Server

Access Gmail, Google Drive, Calendar, Docs, Sheets, and more.

## Installation

```bash
# Install via Homebrew (macOS)
brew install workspace-mcp

# Or via pip
pip install google-workspace-mcp
```

## Setup

### 1. Create OAuth Credentials

1. Go to [Google Cloud Console](https://console.cloud.google.com)
2. Create a new project or select existing
3. Enable required APIs (Gmail, Drive, Calendar, Docs, Sheets)
4. Go to "Credentials" → "Create Credentials" → "OAuth client ID"
5. Choose "Desktop app"
6. Download the credentials

### 2. Add to Claude Code

```bash
claude mcp add -s user google-workspace \
  -e GOOGLE_OAUTH_CLIENT_ID=your_client_id \
  -e GOOGLE_OAUTH_CLIENT_SECRET=your_client_secret \
  -- workspace-mcp --single-user --tool-tier core
```

### 3. Authenticate

First time you use a Google tool, you'll be prompted to authenticate via browser.

## Flags

| Flag | Description |
|------|-------------|
| `--single-user` | Share credentials across all Claude Code sessions |
| `--tool-tier core` | Load core tools only (recommended) |
| `--tool-tier extended` | Load extended tools |
| `--tool-tier complete` | Load all tools |

## Tool Tiers

- **core**: Essential tools for Gmail, Calendar, Drive, Docs
- **extended**: Additional tools for Forms, Tasks, Chat
- **complete**: All available tools

## Multi-Session Support

Use `--single-user` flag to share OAuth credentials across multiple Claude Code sessions. Without it, each session requires separate authentication.

Credentials are stored in `~/.google_workspace_mcp/credentials/`

## Troubleshooting

### Port 8000 in use
OAuth callback requires port 8000. If you see this error:
```bash
pkill -f workspace-mcp
```

### Email mismatch
If Claude uses wrong email, create a symlink:
```bash
ln -s ~/.google_workspace_mcp/credentials/correct@gmail.com.json \
      ~/.google_workspace_mcp/credentials/wrong@gmail.com.json
```

## Tools

| Tool | Description |
|------|-------------|
| `get_doc_content` | Read Google Doc content |
| `create_doc` | Create new Google Doc |
| `search_drive` | Search Drive files |
| `list_calendar_events` | List calendar events |
| `send_email` | Send Gmail |
| `search_email` | Search Gmail |

## Repository

https://github.com/pab1it0/google-workspace-mcp
