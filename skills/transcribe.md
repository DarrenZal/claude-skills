---
allowed-tools:
  - Bash(ssh:*)
  - Read
  - Write
description: Transcribe a YouTube video using the remote transcription server
argument-hint: <youtube-url> [--output file.md] [--whisper]
---

# YouTube Transcription

Transcribe a YouTube video by connecting to the remote transcription server via SSH.

## Arguments

The user provides: $ARGUMENTS

Parse the arguments:
- **YouTube URL**: Required. The YouTube video URL to transcribe.
- **--output <file>**: Optional. Save transcript to a local file.
- **--whisper**: Optional. Force Whisper transcription instead of using YouTube captions.

## Instructions

1. **Extract the YouTube URL** from the arguments. Accept various formats:
   - `https://www.youtube.com/watch?v=VIDEO_ID`
   - `https://youtu.be/VIDEO_ID`
   - `https://youtube.com/shorts/VIDEO_ID`

2. **Run the transcription** on the remote server:

```bash
ssh gaia@37.27.48.12 "cd /home/gaia/transcriptionserver && docker compose exec -T api python /app/transcribe_youtube.py '<URL>' --json"
```

If the user specified `--whisper`, add `--force-whisper` to the command.

3. **Parse the JSON response** which contains:
   - `title`: Video title
   - `duration`: Duration in seconds
   - `source`: Either "captions" (instant) or "whisper" (transcribed)
   - `transcript`: The actual transcript text

4. **Present the results** to the user:
   - Show the video title and duration
   - Indicate whether captions were used (instant) or Whisper transcription
   - Display or save the transcript

5. **If --output was specified**, save the transcript to a markdown file with:
   - YAML frontmatter with title, source URL, duration, date
   - The transcript text

## Example Output File Format

```markdown
---
title: "Video Title Here"
source: https://youtube.com/watch?v=...
duration: 342
transcribed: 2025-01-10
method: captions
---

# Video Title Here

[transcript text here]
```

## Error Handling

- If the SSH connection fails, inform the user to check their SSH configuration
- If transcription fails, show the error message from the server
- If the video is too long (>2 hours), inform the user of the limit

## Notes

- Captions extraction is instant (~1-2 seconds)
- Whisper transcription takes longer depending on video length
- The server uses a residential proxy to access YouTube
