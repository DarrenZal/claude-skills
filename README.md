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

Optionally, copy `CLAUDE.md.example` to `~/.claude/CLAUDE.md` so Claude auto-detects when to use skills.

## Available Skills

### `/meeting-notes`
Create Obsidian meeting notes from any transcript source (MacWhisper, Otter.ai, etc.).

**Requirements:**
- Obsidian vault with `Meetings/` and `Transcripts/` folders
- Claude Code with Obsidian vault MCP server configured
- One or more transcript sources:
  - [MacWhisper](https://goodsnooze.gumroad.com/l/macwhisper) with "Automatically create .whisper file" enabled
  - [Otter.ai](https://otter.ai) with MCP server configured

**Features:**
- Auto-detects transcript source or specify with flags
- Creates separate transcript file in `Transcripts/`
- Creates meeting note in `Meetings/` with link to transcript
- Extracts topics, action items, decisions from transcript
- Supports attendees and project name overrides

**What it creates:**

1. `Transcripts/YYYY-MM-DD <Project> Meeting Transcript.md` - full transcript
2. `Meetings/YYYY-MM-DD <Project> Meeting.md` - structured notes with:
   - Frontmatter (date, project, attendees, topics, transcriptFile link)
   - Summary
   - Key Points
   - Action Items
   - Decisions

**Usage:**
```
/meeting-notes                              # Auto-detect, most recent transcript
/meeting-notes clare                        # Find MacWhisper file matching "clare"
/meeting-notes aaron --otter                # Search Otter for "aaron"
/meeting-notes --macwhisper                 # Force MacWhisper source
/meeting-notes --attendees "Name1, Name2"   # Specify attendees
/meeting-notes --project "ProjectName"      # Override project name
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

## Global CLAUDE.md

Copy `CLAUDE.md.example` to `~/.claude/CLAUDE.md` to enable auto-detection of when to use skills. This tells Claude to automatically invoke `/meeting-notes` when you say things like "make meeting notes for my call with Aaron".

## Contributing

Feel free to submit PRs with new skills or improvements to existing ones.

## License

MIT
