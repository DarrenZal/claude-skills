---
allowed-tools:
  - Bash
  - Read
  - Edit
  - Glob
  - Grep
  - mcp__obsidian-vault__vault_write_note
  - mcp__obsidian-vault__vault_read_note
  - mcp__obsidian-vault__vault_search_notes
  - mcp__otter__otter_list_transcripts
  - mcp__otter__otter_get_transcript
  - mcp__otter__otter_search
  - mcp__google-workspace__get_events
  - mcp__google-workspace__list_calendars
  - mcp__google-workspace__search_gmail_messages
  - mcp__google-workspace__get_gmail_messages_content_batch
  - AskUserQuestion
  - Skill
description: Create Obsidian meeting notes from any transcript source
argument-hint: [name] [--otter] [--macwhisper] [--attendees "..."] [--project "..."]
---

# Meeting Notes from Transcription

Create Obsidian meeting notes from any transcript source. This skill handles:
1. Transcript file in `Transcripts/`
2. Meeting note in `Meetings/` with structure and summary
3. People notes for attendees (with emails from calendar)

**Entity linking (organizations, projects, concepts) is handled by `/process-note`** which runs automatically at the end.

## Configuration

Before using, update these values for your setup:
- `<your-email>` - Your Google email for calendar/Gmail lookup
- `<vault-path>` - Path to your Obsidian vault (default: `~/Documents/Notes/`)
- `<transcripts-path>` - Path to MacWhisper transcripts (default: `~/Documents/AudioTranscriptions/`)

## Arguments

The user provides: $ARGUMENTS

Parse:
- **name**: Optional search term to find transcript (e.g., "clare", "aaron perry")
- **--otter**: Force search Otter.ai
- **--macwhisper**: Force search MacWhisper files
- **--attendees "Name1, Name2"**: Specify attendees
- **--project "Name"**: Override project name
- **--date YYYY-MM-DD**: Override date (defaults to transcript date or today)
- **--email EMAIL**: User's Google email for calendar/email lookup

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
cd <transcripts-path> && ls -t *.whisper | grep -i "<name>" | head -1
```

Or most recent:
```bash
cd <transcripts-path> && ls -t *.whisper | head -1
```

Extract (handle special characters):
```bash
cd <transcripts-path> && ls -t *.whisper | grep -i "<name>" | head -1 | while read f; do cp "$f" /tmp/meeting.whisper; done
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

## Step 3: Look Up Attendee Info from Google (Optional)

If Google Workspace MCP is configured, search Calendar and Gmail for meeting invite to get full attendee names and email addresses.

### 3a: Search Google Calendar

```
get_events(
  user_google_email: "<your-email>",
  query: "<project name or meeting keywords>",
  time_min: "<meeting date>T00:00:00Z",
  time_max: "<meeting date + 1 day>T00:00:00Z",
  detailed: true
)
```

### 3b: Search Gmail (including Spam!)

Meeting invites often land in spam. Search both inbox and spam:

```
search_gmail_messages(
  user_google_email: "<your-email>",
  query: "subject:<project> invite after:<date-7days> before:<date+1day>"
)
```

**Also check spam folder:**
```
search_gmail_messages(
  user_google_email: "<your-email>",
  query: "in:spam <organizer name or project>"
)
```

Then get the message content to extract attendees:
```
get_gmail_messages_content_batch(
  user_google_email: "<your-email>",
  message_ids: [<ids>],
  format: "full"
)
```

### 3c: Extract Attendee Info

From the email invite, extract:
- Full names (transcript often has nicknames like "Ralf" but email has "Ralph Thurm")
- Email addresses
- Organizations (from email domains or signatures)

**If no Google integration:** Extract attendee names from transcript intro (people often introduce themselves).

## Step 4: Create Transcript File

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

## Step 5: Create People Notes

For each attendee, check if `People/<Full Name>.md` exists. If not, create with email (if available):

```
vault_write_note(
  path: "People/<Full Name>",
  content: "# <Full Name>\n\n## Contact\n\n- Email: <email>\n\n## About\n\n<brief context from transcript intro>",
  frontmatter: {
    "@type": "schema:Person",
    "name": "<Full Name>",
    "aliases": ["<nickname from transcript>"],
    "email": "<email>"
  }
)
```

**Note:** Don't add organization affiliations here - `/process-note` will handle entity linking.

## Step 6: Create Meeting Note

**Path:** `Meetings/YYYY-MM-DD <Project> Meeting.md`

### YAML Frontmatter Rules

**IMPORTANT:** Wikilinks ARE supported in YAML but must be:
- Quoted as strings: `"[[People/Name]]"`
- Never inserted mid-text (wrong: `"[[Project]] background"`)
- Complete values only (right: `"[[People/Bill Baue]]"`)

**Frontmatter:**
```yaml
"@type": Meeting
date: YYYY-MM-DD
project: <project>
status: completed
attendees:
  - "[[People/Full Name 1]]"
  - "[[People/Full Name 2|Nickname]]"
topics:
  - <topic 1>
  - <topic 2>
  - <topic 3>
actionItems:
  - "<person>: <action>"
decisions:
  - <decision 1>
nextSteps:
  - <next step>
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

**Do NOT add wikilinks for organizations/projects in the body** - `/process-note` handles this.

## Step 7: Report & Chain to /process-note

Tell the user what was created:
- ✓ Transcript saved: `Transcripts/...`
- ✓ Meeting note created: `Meetings/...`
- ✓ People notes created/updated: X new, Y existing
- Quick summary: X topics, Y action items, Z decisions

Then **IMMEDIATELY invoke /process-note** to handle entity linking:

```
Now running /process-note to link organizations, projects, and concepts...
```

Use the Skill tool:
```
Skill(skill: "process-note", args: "Meetings/YYYY-MM-DD <Project> Meeting.md --apply --create-entities")
```

**This is NOT optional.** Always chain to /process-note for complete entity linking.

## Examples

```
/meeting-notes
→ Finds most recent MacWhisper, checks Gmail for invite, creates files, runs /process-note

/meeting-notes clare
→ Finds MacWhisper file matching "clare"

/meeting-notes aaron --otter
→ Searches Otter for "aaron", creates files, runs /process-note

/meeting-notes --otter --project "YonEarth" --attendees "Aaron Perry, Darren"
→ Uses most recent Otter transcript, overrides project name
```

## Safety Rules

- **Only create NEW files** - never overwrite existing meeting notes or transcripts
- Before writing, check if file exists. If it does, ask user before replacing
- Never modify other vault files during this process
- **Never insert wikilinks into YAML string values mid-text**
- Use Edit tool for small targeted changes, never rewrite entire files

## Error Handling

- If no transcript found: list recent options from both sources
- If multiple matches: ask user to pick
- If transcript already processed: ask if user wants to recreate or skip
- If no calendar/email invite found: extract attendee names from transcript intro
- If emails go to spam: inform user to whitelist the sender

## Lessons Learned

These improvements were added based on real usage:

1. **Check spam folder** - Meeting invites from new contacts often land in spam
2. **Match transcript nicknames to full names** - Transcript says "Ralf", email says "Ralph Thurm"
3. **Wikilinks in YAML must be quoted** - Use `"[[People/Name]]"` not bare `[[People/Name]]`
4. **Don't insert wikilinks mid-string** - `"BKC COP background"` should stay as-is
5. **Create People notes with emails** - Much more useful for future reference
6. **Separate transcript from meeting note** - Transcript goes in `Transcripts/`, meeting note body is for processed notes only
7. **Always chain to /process-note** - Entity linking (orgs, projects, concepts) is handled by the dedicated skill
8. **Don't duplicate /process-note work** - This skill creates structure + People notes; /process-note handles all other entities
