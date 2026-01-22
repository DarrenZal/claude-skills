# Claude Code Skills

A collection of custom skills for [Claude Code](https://claude.ai/code) (Anthropic's CLI tool).

## What are Skills?

Skills are markdown files that teach Claude Code how to perform specific tasks. They're like reusable prompts with instructions, tool permissions, and workflows.

## Installation

Copy any skill file to your Claude Code commands directory:

```bash
# Single skill
cp skills/meeting-notes.md ~/.claude/commands/

# All skills
cp skills/*.md ~/.claude/commands/
```

Then use it in Claude Code:
```
/meeting-notes
```

## Available Skills

### `/meeting-notes`
Create Obsidian meeting notes from MacWhisper transcriptions.

**Requirements:**
- [MacWhisper](https://goodsnooze.gumroad.com/l/macwhisper) with "Automatically create .whisper file" enabled
- Obsidian vault with a Meetings folder
- Claude Code with Obsidian vault MCP server configured

**Features:**
- Extracts transcription from `.whisper` files
- Analyzes transcript for topics, action items, decisions
- Creates structured meeting notes with proper frontmatter
- Can update existing meeting notes

**Usage:**
```
/meeting-notes                              # Process most recent transcription
/meeting-notes "project name"               # Process specific recording
/meeting-notes mehul --attendees "Mehul, Darren"  # With attendees
/meeting-notes --update                     # Update existing note
```

---

### `/transcribe`
Transcribe YouTube videos using a remote transcription server.

**Requirements:**
- SSH access to transcription server

**Features:**
- Uses YouTube captions when available (instant)
- Falls back to Whisper transcription
- Saves to local markdown file

**Usage:**
```
/transcribe https://youtube.com/watch?v=...
/transcribe https://youtu.be/... --output notes.md
/transcribe <url> --whisper    # Force Whisper instead of captions
```

## Creating Your Own Skills

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

Instructions for Claude Code to follow when this skill is invoked.

## Arguments

The user provides: $ARGUMENTS

## Instructions

1. Step one
2. Step two
...
```

### Frontmatter Fields

| Field | Description |
|-------|-------------|
| `allowed-tools` | List of tools the skill can use |
| `description` | One-line description |
| `argument-hint` | Shows usage hint (e.g., `<url> [--flag]`) |

### Variables

- `$ARGUMENTS` - User input after the skill name

## Contributing

Feel free to submit PRs with new skills or improvements to existing ones.

## License

MIT
