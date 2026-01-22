# Obsidian Vault MCP Server

Read, write, and search notes in your Obsidian vault.

## Features

- Read/write markdown notes
- Search by content, frontmatter, or entity type
- Find backlinks and relationships
- Fuzzy name matching
- Meeting prep with attendee context

## Installation

```bash
git clone https://github.com/DarrenZal/obsidian-vault-mcp.git
cd obsidian-vault-mcp
npm install
npm run build
```

## Setup

```bash
claude mcp add -s user obsidian-vault \
  -e OBSIDIAN_VAULT_PATH=/path/to/your/vault \
  -- node /path/to/obsidian-vault-mcp/dist/index.js
```

## Tools

| Tool | Description |
|------|-------------|
| `vault_read_note` | Read note content with frontmatter |
| `vault_write_note` | Create or update a note |
| `vault_search_notes` | Search by query string |
| `vault_list_notes` | List notes by folder or entity type |
| `vault_get_entity` | Get entity by type and name |
| `vault_find_backlinks` | Find notes linking to a note |
| `vault_query_frontmatter` | Query by YAML frontmatter fields |
| `vault_find_person` | Find all info about a person |
| `vault_prep_meeting` | Prepare for meeting with attendee context |

## Usage Examples

```
Read a note:
→ vault_read_note(path: "People/John Smith")

Search notes:
→ vault_search_notes(query: "project timeline", searchContent: true)

Find person info:
→ vault_find_person(name: "Clare Attwell")

Prep for meeting:
→ vault_prep_meeting(attendees: ["Clare Attwell", "John Smith"], project: "MyProject")
```

## Note Structure

The MCP works best with structured notes:

```markdown
---
"@type": Person
name: John Smith
affiliation: Acme Corp
---

# John Smith

Works at [[Acme Corp]].
```

## Entity Types

- `Person` - People notes
- `Organization` - Company/org notes
- `Meeting` - Meeting notes
- `Project` - Project notes

## Repository

https://github.com/DarrenZal/obsidian-vault-mcp
