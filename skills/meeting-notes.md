---
allowed-tools:
  - Bash
  - Read
  - Write
  - mcp__obsidian-vault__vault_write_note
  - mcp__obsidian-vault__vault_read_note
  - mcp__obsidian-vault__vault_search_notes
description: Create Obsidian meeting notes from MacWhisper transcriptions
argument-hint: [recording-name] [--attendees "Name1, Name2"]
---

# Meeting Notes from MacWhisper Transcription

Create Obsidian meeting notes from MacWhisper transcriptions. Keep it fast and simple.

## Arguments

The user provides: $ARGUMENTS

- **recording-name**: Optional. Partial name to match. If omitted, use most recent .whisper file.
- **--attendees "Name1, Name2"**: Optional. Comma-separated attendees.
- **--project "Name"**: Optional. Override project name.

## Instructions

### 1. Find and extract the transcription

Use glob pattern to handle MacWhisper's special characters in filenames:

```bash
cd ~/Documents/AudioTranscriptions && ls -t *.whisper | grep -i "<name>" | head -1 | while read f; do cp "$f" /tmp/meeting.whisper; done
```

If no name given, get most recent:
```bash
cd ~/Documents/AudioTranscriptions && ls -t *.whisper | head -1 | while read f; do cp "$f" /tmp/meeting.whisper; done
```

Extract transcript:
```bash
cd /tmp && unzip -o meeting.whisper metadata.json && cat metadata.json | python3 -c "
import json, sys
data = json.load(sys.stdin)
print('FILENAME:', data.get('originalMediaFilename', 'Unknown'))
texts = [t.get('text', '') for t in data.get('transcripts', [])]
print('---TRANSCRIPT---')
print(' '.join(texts))
"
```

### 2. Parse the date and project name

From filename like "Jan 21 12.23.52 PM Clare Atwell":
- Date: 2026-01-21 (convert "Jan 21" to YYYY-MM-DD using current year)
- Project: "Clare Atwell" (or use --project override)

### 3. Save the transcript as a separate file

Create: `Transcripts/YYYY-MM-DD <Project> Meeting Transcript.md`

```markdown
# <Project> Meeting Transcript - YYYY-MM-DD

Source: MacWhisper
Original file: <filename>

---

<full transcript text>
```

### 4. Create the meeting note

Create: `Meetings/YYYY-MM-DD <Project> Meeting.md`

**Frontmatter:**
```yaml
"@type": Meeting
date: YYYY-MM-DD
project: <project>
status: completed
attendees:
  - "[[Attendee Name]]"
topics:
  - <2-3 main topics from quick scan>
transcriptFile: "[[Transcripts/YYYY-MM-DD <Project> Meeting Transcript]]"
transcriptSource: MacWhisper
originalFile: <filename>
```

**Body:**
```markdown
# <Project> Meeting - YYYY-MM-DD

## Summary
<2-3 sentence summary - keep brief>

## Key Points
- <bullet points of main discussion items>

## Action Items
- <any clear action items mentioned>
```

### 5. Report back

Tell the user:
- Meeting note created at: `Meetings/...`
- Transcript saved at: `Transcripts/...`
- Brief summary of what was discussed

## Keep it fast

- Don't over-analyze - a quick scan for topics is enough
- Save the full transcript separately so it's available for deeper review
- The user can always ask for more detailed analysis later
