# Process Note: Entity Extraction & Linking

Extract entities from a document and link them to your Obsidian vault, with optional backend integration for knowledge base storage.

## Usage

```
/process-note [path]
/process-note Articles/My Research.md
/process-note --create-entities      # Also create new entity files
/process-note --apply                # Apply changes (not just preview)
/process-note --backend              # Store entities in personal KOI backend
```

## Arguments

- `path` - Path to the document to process (relative to vault root)
- `--create-entities` - Create new entity files for unmatched entities
- `--apply` - Apply wikilinks to the document (otherwise preview only)
- `--backend` - Ingest entities to personal KOI backend for deduplication and storage
- `--attendees "Name1, Name2"` - Override attendee detection
- `--project "ProjectName"` - Set project context

## Workflow

This skill orchestrates the full entity linking workflow:

1. **Load Document** - Read the target document from your vault
2. **Build Entity Index** - Scan People/, Organizations/, Projects/, Locations/ folders
3. **Extract Entities** - Claude identifies entities in the document (no API cost)
4. **Backend Integration** (optional) - Send to personal KOI backend for:
   - Deduplication against existing knowledge base
   - Canonical URI assignment
   - Persistent storage in PostgreSQL
5. **Resolve Entities** - Match against vault notes (exact, fuzzy, or new)
6. **Preview Changes** - Show proposed wikilinks and frontmatter
7. **Apply (if requested)** - Insert wikilinks and update frontmatter

## Entity Types Extracted

- **People** - Named individuals (authors, researchers, experts)
- **Organizations** - Companies, agencies, NGOs, universities, First Nations
- **Locations** - Geographic places, bodies of water, regions
- **Projects** - Named initiatives, programs, research projects
- **Concepts** - Key topics, technical terms, central themes

## Resolution Tiers

1. **Exact Match** - Normalized name matches existing vault entity
2. **Alias Match** - Name matches an alias defined in frontmatter
3. **Fuzzy Match** - Jaro-Winkler similarity above threshold
4. **KB Match** (with backend) - Entity exists in personal knowledge base
5. **New Entity** - No match found, suggest creating new note

## Example Output

```markdown
## Processing Result: Articles/Salish Sea Herring.md

### Statistics
- Entities extracted: 15
- Entities resolved: 8
- New entities created: 5
- Stored in KB: 15
- Wikilinks to add: 42

### Extracted Entities

**Organizations:**
- DFO → ✅ EXACT → [[Organizations/DFO]]
  URI: orn:personal-koi.entity:organization-dfo-abc123
- Songhees Nation → 🆕 NEW → [[Organizations/Songhees Nation]]
  URI: orn:personal-koi.entity:organization-songhees-nation-def456

**People:**
- Jake Dingwall → 🆕 NEW → [[People/Jake Dingwall]]
- Amanda Bates → ⚡ FUZZY (92%) → [[People/Amanda Bates]]

### Suggested Frontmatter
```yaml
"@type": Article
topics:
  - Herring fishery
  - Marine ecology
mentions:
  - "[[Organizations/DFO]]"
  - "[[People/Jake Dingwall]]"
```

### New Entity Files to Create
- **People/Jake Dingwall.md** (Person)
  > Herring restoration advocate, mentioned in context of...
```

## Backend Integration (Personal KOI-net)

When the backend is enabled (`--backend` flag), entities are:
- Deduplicated against your personal knowledge base
- Assigned stable canonical URIs (orn:personal-koi.entity:...)
- Stored in PostgreSQL with pgvector embeddings
- Linked to source documents via RIDs

The backend runs locally on port 8351. To start it:
```bash
launchctl load ~/Library/LaunchAgents/com.personal.koi-processor.plist
```

## Future: Selective Sharing with Regen KOI

Once personal KOI-net is working:
- Add "share to Regen" flag on entity creation
- Batch export selected entities to Regen KB
- Implement KOI-net peering protocol for bidirectional sync

---

$ARGUMENTS

Process the note at the given path. If no path provided, ask the user which document to process.

## CRITICAL SAFETY RULES

**DO NOT rewrite entire files.** This causes data loss due to Read tool truncation limits.

- NEVER use `vault_write_note` to rewrite an existing document
- NEVER use the Write tool on vault files
- Only use small, targeted Edit operations to insert wikilinks
- If a file is too large to safely edit, warn the user and skip it

**Steps:**

1. Use `vault_extract_entities` to build the extraction prompt and load vault context

2. Analyze the document content and extract entities following the prompt guidelines. Return a JSON object with this structure:
   ```json
   {
     "entities": [
       {"name": "Entity Name", "type": "Person|Organization|Location|Project|Concept", "mentions": ["mention1", "mention2"], "confidence": 0.95, "context": "Brief description"}
     ],
     "topics": ["topic1", "topic2"],
     "documentType": "Article|Report|Meeting Notes|Research Paper|Other"
   }
   ```

3. If `--backend` flag is set or user wants backend integration:
   - Use `vault_ingest_extraction` to send entities to the personal KOI backend
   - This returns deduplicated entities with canonical URIs

4. Otherwise, use `vault_process_extraction` with `preview: true` (default) to resolve against vault only

5. Show the user the preview - **DO NOT auto-apply changes**

6. If user explicitly confirms they want changes applied:
   - Use the **Edit tool** with small, targeted replacements (one entity at a time)
   - For each entity mention, replace `EntityName` with `[[EntityName]]`
   - Create new entity files if `--create-entities` is set using `vault_write_note` (new files only)
   - Update frontmatter using targeted Edit operations, not full rewrites
