---
allowed-tools:
  - Bash
  - Read
  - Write
  - mcp__obsidian-vault__vault_write_note
  - mcp__obsidian-vault__vault_read_note
  - mcp__obsidian-vault__vault_search_notes
  - mcp__obsidian-vault__vault_list_notes
description: Create Obsidian meeting notes from MacWhisper transcriptions
argument-hint: [recording-name] [--attendees "Name1, Name2"] [--update]
---

# Meeting Notes from MacWhisper Transcription

Process a MacWhisper transcription and create/update structured Obsidian meeting notes.

## Schema Reference

The meeting note schema is defined in the Obsidian vault at:
`Templates/schema-meeting.md`

Always read this schema first to ensure the output matches the expected structure.

## Arguments

The user provides: $ARGUMENTS

Parse the arguments:
- **recording-name**: Optional. Partial name to match (e.g., "regen cascadia"). If omitted, use the most recent .whisper file.
- **--attendees "Name1, Name2"**: Optional. Comma-separated list of attendees.
- **--update**: Optional. If provided, look for an existing meeting note for the same project/date and update it with the transcript rather than creating a new one.
- **--project "Project Name"**: Optional. Override the project name instead of inferring from filename.

## Configuration

- **Transcriptions folder**: `~/Documents/AudioTranscriptions/`
- **Obsidian vault**: `~/Documents/Notes/`
- **Meetings folder**: `Meetings/`
- **Schema file**: `Templates/schema-meeting.md`

## Instructions

### 0. Read the schema

First, read the meeting schema to ensure consistency:
```
vault_read_note("Templates/schema-meeting")
```

### 1. Find the transcription file

If a recording name was provided:
```bash
ls -t ~/Documents/AudioTranscriptions/*.whisper | grep -i "<recording-name>" | head -1
```

If no name provided, get the most recent:
```bash
ls -t ~/Documents/AudioTranscriptions/*.whisper | head -1
```

### 2. Extract the transcription

Copy the .whisper file and extract:
```bash
cd ~/Documents/AudioTranscriptions && cp <pattern>*.whisper /tmp/meeting.whisper
cd /tmp && unzip -o meeting.whisper metadata.json
```

Parse the metadata.json to extract:
- `originalMediaFilename`: The recording name
- `transcripts[].text`: The transcript segments (join them together)
- Recording date from the filename (format: "Mon DD H.MM.SS AM/PM")

Use this Python snippet to extract:
```bash
cat /tmp/metadata.json | python3 -c "
import json, sys
data = json.load(sys.stdin)
print('FILENAME:', data.get('originalMediaFilename', 'Unknown'))
print('LANGUAGE:', data.get('detectedLanguageRaw', 'Unknown'))
print('---TRANSCRIPT---')
for t in data.get('transcripts', []):
    print(t.get('text', ''))
"
```

### 3. Check for existing meeting note (if --update or matching note exists)

Search for an existing meeting note:
```
vault_search_notes(query="<project name> <date>", entityType="Meeting")
```

Or list recent meetings:
```
vault_list_notes(folder="Meetings", limit=10)
```

If a matching note exists:
- Read it with `vault_read_note`
- Preserve existing frontmatter fields
- Add/update: transcriptSource, originalFile, notes (append transcript summary)
- Add the full transcript to the body

### 4. Analyze the transcript

From the transcript text, identify:
- **Key topics** discussed
- **Action items** (look for phrases like "I'll", "we should", "need to", "will do", "follow up", "action item")
- **Decisions** made (look for "decided", "agreed", "we'll go with", "the plan is")
- **Project context** (if identifiable from content or filename)
- **Attendees** (if mentioned by name in the transcript)

### 5. Create or update the Obsidian meeting note

Use `vault_write_note` with the path:
`Meetings/YYYY-MM-DD <Project Name> Meeting.md`

**Frontmatter** (follow schema exactly):
```yaml
"@type": Meeting
date: YYYY-MM-DD
project: <project name>
meetingType: <inferred or blank>
status: completed
location: <if known, otherwise blank>
attendees:
  - "[[Attendee Name]]"
agenda:
  - <if known from previous meeting>
topics:
  - Topic 1
  - Topic 2
notes: |
  <summary notes organized by topic>
actionItems:
  - Action item 1
  - Action item 2
decisions:
  - Decision 1
nextSteps:
  - Next step 1
transcriptFile: "[[Transcripts/YYYY-MM-DD <Project> Meeting Transcript]]"
transcriptSource: MacWhisper
originalFile: <original filename>
```

**Body**:
```markdown
# <Project Name> Meeting - YYYY-MM-DD

## Summary
<2-3 sentence summary of the meeting>

## Notes
<Organized notes from the transcript, grouped by topic>

## Full Transcript
<details>
<summary>Click to expand full transcript</summary>

<full transcript text>

</details>
```

### 6. Report back

Tell the user:
- Whether a new note was created or existing one updated
- The path to the note in Obsidian
- Summary: number of topics, action items, decisions captured
- Any attendees identified

## Example Usage

- `/meeting-notes` - Process the most recent transcription
- `/meeting-notes regen cascadia` - Process a specific recording
- `/meeting-notes mehul --attendees "Mehul, Darren"` - With attendees specified
- `/meeting-notes --update` - Update existing meeting note with latest transcript
- `/meeting-notes gaia --project "GAIA AI"` - Override project name

## Error Handling

- If no .whisper files found, inform the user to check MacWhisper settings ("Automatically create .whisper file" should be enabled)
- If the specified recording isn't found, list the 5 most recent recordings to choose from
- If transcript extraction fails, show the error and suggest manual export from MacWhisper
- If the meeting note already exists and --update wasn't specified, ask whether to update or create new
