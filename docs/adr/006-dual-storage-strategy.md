# ADR-006: Dual Storage Strategy (SQLite + Markdown Cache)

## Status

Accepted (2024)

## Context

Feedback data needs to be:
- **Queryable**: Fast searches and aggregations (SQLite)
- **Human-Readable**: Easy to browse and review (Markdown)
- **Portable**: Easy to backup and share
- **Redundant**: No single point of failure

Single storage approach has limitations:
- SQLite alone: Not human-readable, requires tools to inspect
- Markdown alone: Slow queries, no full-text search, no aggregations

## Decision

Implement a **dual storage strategy** where feedback is stored in both:

### 1. SQLite Database (Primary)

**Purpose**: Fast queries, aggregations, full-text search

**Location**: `~/.claude/claudarity.db`

**Schema**:
```sql
CREATE TABLE feedback_entries (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    ts TEXT NOT NULL,
    project TEXT,
    pattern TEXT,
    type TEXT CHECK(type IN ('win', 'loss')),
    user_message TEXT,
    context_summary TEXT,
    ai_summary TEXT,
    cache_file TEXT  -- Reference to markdown file
);

CREATE VIRTUAL TABLE feedback_fts USING fts5(
    feedback_id,
    user_message,
    context_summary,
    ai_summary
);
```

**Use Cases**:
- `/baseline` command (aggregate statistics)
- `/gomemory` command (full-text search)
- Confidence score calculations
- Template evolution analysis

### 2. Markdown Cache Files (Secondary)

**Purpose**: Human-readable backup, easy browsing

**Location**: `/Volumes/DevDrive/Cache/feedback/`

**Format**:
```markdown
# Win: great job

**Time:** 2024-01-15 10:30:00
**Project:** my-app
**Pattern:** great job

## AI Summary
User successfully implemented authentication with JWT tokens.
The implementation follows best practices and includes proper
error handling.

## Quick Context
✏️ auth.ts, ✏️ middleware.ts, 📖 config.ts

## User Message
great job
```

**Filename Convention**: `{type}-{timestamp}.md`
- Example: `win-2024-01-15-10-30-00.md`

**Use Cases**:
- Manual review and browsing
- Backup and archival
- Sharing feedback examples
- Debugging and auditing

### Synchronization

```bash
# Write to both simultaneously
# 1. Insert into SQLite
sqlite3 "$DB" "INSERT INTO feedback_entries (...) VALUES (...);"
last_id=$(sqlite3 "$DB" "SELECT last_insert_rowid();")

# 2. Create markdown file
cat > "$cache_file" <<EOF
# Win: $pattern
...
EOF

# 3. Update SQLite with cache file reference
sqlite3 "$DB" "UPDATE feedback_entries SET cache_file = '$cache_file' WHERE rowid = $last_id;"
```

## Consequences

### Positive

- **Best of Both Worlds**: Fast queries + human readability
- **Redundancy**: Data loss requires both systems to fail
- **Flexibility**: Can query SQLite or browse markdown
- **Debugging**: Easy to inspect markdown files
- **Portability**: Markdown files easy to share/backup
- **Recovery**: Can rebuild SQLite from markdown if needed
- **Transparency**: Users can see what's being stored

### Negative

- **Storage Overhead**: Data stored twice (~2x disk usage)
- **Sync Complexity**: Must keep both in sync
- **Write Overhead**: Two write operations per feedback
- **Consistency Risk**: Could get out of sync if one fails
- **Maintenance**: Two systems to maintain

### Mitigations

- **Atomic Writes**: Use transactions to ensure consistency
- **Error Handling**: Graceful degradation if one system fails
- **Reconciliation**: Script to sync SQLite ↔ Markdown
- **Compression**: Compress old markdown files
- **Cleanup**: Automated cleanup of old cache files

## Alternatives Considered

### 1. SQLite Only
- **Pros**: Single source of truth, simpler, less storage
- **Cons**: Not human-readable, requires tools to inspect
- **Rejected**: Transparency and debugging too important

### 2. Markdown Only
- **Pros**: Human-readable, simple, portable
- **Cons**: Slow queries, no full-text search, no aggregations
- **Rejected**: Performance requirements not met

### 3. SQLite + JSON Export
- **Pros**: Structured, machine-readable
- **Cons**: Not human-friendly, requires parsing
- **Rejected**: Markdown more accessible

### 4. SQLite + HTML Export
- **Pros**: Rich formatting, viewable in browser
- **Cons**: More complex, harder to edit
- **Rejected**: Markdown simpler and more portable

### 5. Git-Based Storage
- **Pros**: Version control, history tracking
- **Cons**: Complex, slow, overkill
- **Rejected**: Too complex for this use case

## Implementation Notes

### Write Path

```bash
# 1. Prepare data
ts_iso=$(date -u '+%Y-%m-%dT%H:%M:%SZ')
ai_summary=$(generate_summary "$transcript_file")

# 2. Write to SQLite (with transaction)
sqlite3 "$DB" <<SQL
BEGIN TRANSACTION;
INSERT INTO feedback_entries (ts, project, pattern, type, user_message, context_summary, ai_summary)
VALUES ('$ts_iso', '$project', '$pattern', 'win', '$user_message', '$context_summary', '$ai_summary');
COMMIT;
SQL

# 3. Get last insert ID
last_id=$(sqlite3 "$DB" "SELECT last_insert_rowid();")

# 4. Create markdown file
cache_file="/Volumes/DevDrive/Cache/feedback/win-${ts_iso}.md"
cat > "$cache_file" <<EOF
# Win: $pattern
...
EOF

# 5. Update SQLite with cache file reference
sqlite3 "$DB" "UPDATE feedback_entries SET cache_file = '$cache_file' WHERE rowid = $last_id;"
```

### Read Path

```bash
# Query SQLite for fast search
results=$(sqlite3 "$DB" "SELECT * FROM feedback_entries WHERE ...")

# Or browse markdown files
ls -lt /Volumes/DevDrive/Cache/feedback/ | head -20
cat /Volumes/DevDrive/Cache/feedback/win-2024-01-15-10-30-00.md
```

### Reconciliation Script

```bash
#!/usr/bin/env bash
# Sync SQLite and Markdown

# Find markdown files not in SQLite
for md_file in /Volumes/DevDrive/Cache/feedback/*.md; do
  if ! sqlite3 "$DB" "SELECT 1 FROM feedback_entries WHERE cache_file = '$md_file';" | grep -q 1; then
    echo "Missing in SQLite: $md_file"
    # Parse markdown and insert into SQLite
  fi
done

# Find SQLite entries without markdown files
sqlite3 "$DB" "SELECT cache_file FROM feedback_entries WHERE cache_file IS NOT NULL;" | \
  while read cache_file; do
    if [ ! -f "$cache_file" ]; then
      echo "Missing markdown: $cache_file"
      # Regenerate markdown from SQLite
    fi
  done
```

## Storage Characteristics

### Disk Usage

- **SQLite**: ~1KB per feedback entry
- **Markdown**: ~2KB per feedback entry (includes formatting)
- **Total**: ~3KB per entry
- **Example**: 10,000 entries = ~30MB

### Performance

- **SQLite Write**: ~1ms
- **Markdown Write**: ~2ms
- **Total Write Time**: ~3ms (acceptable)
- **SQLite Query**: <10ms
- **Markdown Browse**: Instant (file system)

## Backup Strategy

### SQLite Backup

```bash
# Daily backup
cp ~/.claude/claudarity.db ~/.claude/backups/claudarity-$(date +%Y%m%d).db

# Or use SQLite backup command
sqlite3 ~/.claude/claudarity.db ".backup ~/.claude/backups/claudarity-$(date +%Y%m%d).db"
```

### Markdown Backup

```bash
# Markdown files already on separate volume
# Additional backup to cloud/external drive
rsync -av /Volumes/DevDrive/Cache/feedback/ ~/Backups/claudarity-feedback/
```

### Recovery

```bash
# Rebuild SQLite from markdown (if SQLite corrupted)
~/.claude/scripts/rebuild-db-from-markdown.sh

# Regenerate markdown from SQLite (if markdown lost)
~/.claude/scripts/regenerate-markdown-cache.sh
```

## Future Enhancements

- **Compression**: Compress old markdown files (gzip)
- **Archival**: Move old entries to archive storage
- **Encryption**: Encrypt sensitive feedback data
- **Cloud Sync**: Optional sync to cloud storage
- **Version Control**: Track changes to feedback entries
- **Export Formats**: Export to PDF, HTML, JSON

## References

- `hooks/log-feedback.sh`: Dual write implementation
- `scripts/rebuild-db-from-markdown.sh`: Recovery script
- SQLite Backup: https://www.sqlite.org/backup.html

## Review Date

Review after 6 months to assess:
- Storage overhead impact
- Sync consistency issues
- Backup/recovery effectiveness
- User satisfaction with dual approach
