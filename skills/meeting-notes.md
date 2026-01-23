---
allowed-tools:
  - Bash
  - Read
  - Write
  - mcp__obsidian-vault__vault_write_note
  - mcp__obsidian-vault__vault_read_note
  - mcp__obsidian-vault__vault_search_notes
  - mcp__otter__otter_list_transcripts
  - mcp__otter__otter_get_transcript
  - mcp__otter__otter_search
  - AskUserQuestion
description: Create Obsidian meeting notes from any transcript source
argument-hint: [name] [--otter] [--macwhisper] [--attendees "..."] [--project "..."]
---

# Meeting Notes from Transcription

Create Obsidian meeting notes from any transcript source. Always creates:
1. Transcript file in `Transcripts/`
2. Meeting note in `Meetings/` with link to transcript

## Arguments

The user provides: $ARGUMENTS

Parse:
- **name**: Optional search term to find transcript (e.g., "clare", "aaron perry")
- **--otter**: Force search Otter.ai
- **--macwhisper**: Force search MacWhisper files
- **--attendees "Name1, Name2"**: Specify attendees
- **--project "Name"**: Override project name
- **--date YYYY-MM-DD**: Override date (defaults to transcript date or today)

## Step 1: Determine transcript source

**If --otter flag or user mentioned "otter":**
→ Go to Otter flow

**If --macwhisper flag or user mentioned "macwhisper" or "recording":**
→ Go to MacWhisper flow

**If no flag specified:**
1. Check for recent MacWhisper files (last 24 hours)
2. If found, ask user: "Found recent MacWhisper recording '<name>'. Use this, or search Otter?"
3. If no recent MacWhisper, check Otter for recent transcripts
4. Let user confirm which transcript to use

## Step 2a: MacWhisper Flow

Find the file:
```bash
cd ~/Documents/AudioTranscriptions && ls -t *.whisper | grep -i "<name>" | head -1
```

Or most recent:
```bash
cd ~/Documents/AudioTranscriptions && ls -t *.whisper | head -1
```

Extract (handle special characters):
```bash
cd ~/Documents/AudioTranscriptions && ls -t *.whisper | grep -i "<name>" | head -1 | while read f; do cp "$f" /tmp/meeting.whisper; done
cd /tmp && unzip -o meeting.whisper metadata.json
cat metadata.json | python3 -c "
import json, sys
data = json.load(sys.stdin)
print('FILENAME:', data.get('originalMediaFilename', ''))
texts = [t.get('text', '') for t in data.get('transcripts', [])]
print('---TRANSCRIPT---')
print(' '.join(texts))
"
```

Parse date from filename (e.g., "Jan 21 12.23.52 PM Clare Atwell"):
- Extract month/day, convert to YYYY-MM-DD using current year
- Project name = remainder after timestamp

## Step 2b: Otter Flow

List recent or search:
```
otter_list_transcripts(limit: 10)
```
or
```
otter_search(query: "<name>", limit: 5)
```

Get the transcript:
```
otter_get_transcript(transcript_id: "<id>")
```

Parse date from Otter's `created` timestamp (Unix epoch → YYYY-MM-DD).

## Step 3: Create Transcript File

**Path:** `Transcripts/YYYY-MM-DD <Project> Meeting Transcript.md`

**Content:**
```markdown
# <Project> Meeting Transcript - YYYY-MM-DD

**Source:** <MacWhisper / Otter.ai>
**Original file:** <filename or Otter title>
**Date:** YYYY-MM-DD

---

<full transcript text>
```

Use `vault_write_note` to create:
```
vault_write_note(
  path: "Transcripts/YYYY-MM-DD <Project> Meeting Transcript",
  content: <markdown above>,
  frontmatter: {
    "@type": "Transcript",
    "date": "YYYY-MM-DD",
    "source": "<MacWhisper/Otter>",
    "originalFile": "<filename>"
  }
)
```

## Step 4: Create Meeting Note

**Path:** `Meetings/YYYY-MM-DD <Project> Meeting.md`

**Frontmatter:**
```yaml
"@type": Meeting
date: YYYY-MM-DD
project: <project>
status: completed
attendees:
  - "[[Name1]]"
  - "[[Name2]]"
topics:
  - <topic 1>
  - <topic 2>
  - <topic 3>
transcriptFile: "[[Transcripts/YYYY-MM-DD <Project> Meeting Transcript]]"
transcriptSource: <MacWhisper/Otter>
```

**Body (REQUIRED - do not leave empty):**
```markdown
# <Project> Meeting - YYYY-MM-DD

## Summary
<2-3 sentences summarizing the meeting>

## Key Points
- <main point 1>
- <main point 2>
- <main point 3>

## Action Items
- [ ] <action 1>
- [ ] <action 2>

## Decisions
- <decision 1>
- <decision 2>
```

Quick-scan the transcript for:
- **Topics**: Main subjects discussed (3-5 items)
- **Action items**: Look for "I'll", "we should", "need to", "will", "todo", "action"
- **Decisions**: Look for "decided", "agreed", "we'll go with", "the plan is"

Keep analysis brief - the full transcript is linked for details.

## Step 5: Report Back

Tell the user:
- ✓ Transcript saved: `Transcripts/...`
- ✓ Meeting note created: `Meetings/...`
- Quick summary: X topics, Y action items, Z decisions
- Attendees linked

## Examples

```
/meeting-notes
→ Finds most recent MacWhisper, creates both files

/meeting-notes clare
→ Finds MacWhisper file matching "clare"

/meeting-notes aaron --otter
→ Searches Otter for "aaron", creates both files

/meeting-notes --otter --project "YonEarth" --attendees "Aaron Perry, Darren"
→ Uses most recent Otter transcript, overrides project name
```

## Safety Rules

- **Only create NEW files** - never overwrite existing meeting notes or transcripts
- Before writing, check if file exists. If it does, ask user before replacing
- Never modify other vault files during this process

## Error Handling

- If no transcript found: list recent options from both sources
- If multiple matches: ask user to pick
- If transcript already processed: ask if user wants to recreate or skip
