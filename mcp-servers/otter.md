# Otter.ai MCP Server

Search and retrieve meeting transcripts from Otter.ai.

## Features

- Full-text search across all transcripts
- Date filtering (today, yesterday, this week, date ranges)
- Speaker identification
- Human-readable timestamps

## Installation

### Option 1: Use the published package

```bash
pip install otter-mcp
```

```bash
claude mcp add -s user otter \
  -e OTTER_EMAIL=your@email.com \
  -e OTTER_PASSWORD=your_password \
  -- python -m otter_mcp
```

### Option 2: Local development version

```bash
git clone https://github.com/DarrenZal/otter-mcp.git
cd otter-mcp
python -m venv env
source env/bin/activate
pip install -e .
```

```bash
claude mcp add -s user otter \
  -e OTTER_EMAIL=your@email.com \
  -e OTTER_PASSWORD=your_password \
  -- /path/to/otter-mcp/env/bin/python -m otter_mcp
```

## Tools

| Tool | Description |
|------|-------------|
| `otter_search` | Search transcripts by keyword with optional date filter |
| `otter_list_transcripts` | List recent transcripts |
| `otter_get_transcript` | Get full transcript text by ID |
| `otter_get_user` | Get account info |

## Usage Examples

```
Search for meetings with John from last week
→ otter_search(query: "John", date_filter: "last week")

List today's meetings
→ otter_list_transcripts(date_filter: "today")

Get specific transcript
→ otter_get_transcript(transcript_id: "abc123")
```

## Date Filters

- `today` - meetings from today
- `yesterday` - meetings from yesterday
- `this week` - meetings from this week
- `last week` - meetings from last week
- `2025-01-20` - specific date
- `2025-01-15 to 2025-01-20` - date range

## Repository

https://github.com/DarrenZal/otter-mcp
