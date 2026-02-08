# ADR-002: SQLite as Primary Storage Layer

## Status

Accepted (2024)

## Context

The memory system needs persistent storage for:
- Feedback logs (wins/losses)
- Conversation context
- Code preferences
- Session metadata
- Template evolution data
- Terminal activity

Requirements:
- **Local-first**: All data stays on user's machine (privacy)
- **Fast queries**: Sub-second context retrieval
- **ACID compliance**: Concurrent access from multiple hooks/scripts
- **Full-text search**: Semantic search across conversations
- **Low overhead**: Minimal dependencies, works out-of-box
- **Portable**: Works across macOS/Linux

## Decision

Use **SQLite** as the primary storage layer with the following design:

### Database Schema

```sql
-- Main feedback table
CREATE TABLE feedback_entries (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    ts TEXT NOT NULL,
    project TEXT,
    pattern TEXT,
    type TEXT CHECK(type IN ('win', 'loss')),
    user_message TEXT,
    context_summary TEXT,
    ai_summary TEXT,
    cache_file TEXT
);

-- Full-text search index
CREATE VIRTUAL TABLE feedback_fts USING fts5(
    feedback_id,
    user_message,
    context_summary,
    ai_summary
);

-- Additional tables for context, preferences, sessions, etc.
```

### Key Features Used

1. **FTS5 (Full-Text Search)**
   - Enables semantic search across feedback and context
   - Supports ranking and relevance scoring
   - Fast even with large datasets

2. **ACID Transactions**
   - Multiple hooks can write concurrently safely
   - No data corruption from parallel access
   - Automatic rollback on errors

3. **Single File Database**
   - `~/.claude/claudarity.db`
   - Easy backup (just copy the file)
   - No server process required

4. **Built-in Functions**
   - Date/time functions for temporal queries
   - JSON functions for structured data
   - Aggregate functions for statistics

### Access Patterns

- **Writes**: Async, buffered through background hooks
- **Reads**: Direct queries from scripts/commands
- **Maintenance**: Periodic VACUUM and index optimization
- **Backup**: File-based snapshots

## Consequences

### Positive

- **Zero Configuration**: SQLite included with macOS/Linux
- **Privacy**: All data local, no cloud dependencies
- **Performance**: Fast queries even with 100k+ records
- **Reliability**: Battle-tested, ACID-compliant
- **Portability**: Single file, easy to backup/restore
- **Tooling**: Standard SQL, many tools available
- **Concurrent Access**: Multiple processes can read/write safely
- **Full-Text Search**: Built-in FTS5 for semantic queries

### Negative

- **Single Machine**: No built-in sync across devices
- **File Locking**: Can have contention under heavy concurrent writes
- **Size Growth**: Database grows over time (mitigated by VACUUM)
- **No Built-in Replication**: Manual backup required
- **Limited Concurrency**: Write locks can block (rare in practice)

### Mitigations

- **Background Writes**: Hooks fork to background to avoid blocking
- **Connection Pooling**: Reuse connections where possible
- **Periodic Maintenance**: Automated VACUUM and cleanup scripts
- **Backup Strategy**: Session logs provide redundancy
- **Error Handling**: Graceful degradation if DB unavailable

## Alternatives Considered

### 1. Plain Text Files (JSON/JSONL)
- **Pros**: Simple, human-readable, no dependencies
- **Cons**: No ACID, slow queries, no full-text search, concurrent access issues
- **Rejected**: Doesn't scale, unsafe concurrent access

### 2. PostgreSQL
- **Pros**: More powerful, better concurrency, replication
- **Cons**: Requires server process, complex setup, overkill for local use
- **Rejected**: Too heavy for local-first tool

### 3. Redis
- **Pros**: Fast, good for caching
- **Cons**: In-memory (persistence optional), requires server, no SQL
- **Rejected**: Not suitable for persistent storage

### 4. Embedded Key-Value Store (LevelDB, RocksDB)
- **Pros**: Fast writes, good for logs
- **Cons**: No SQL, no full-text search, more complex queries
- **Rejected**: SQL queries too valuable for this use case

### 5. MongoDB/Document DB
- **Pros**: Flexible schema, good for JSON
- **Cons**: Requires server, heavier than SQLite, no FTS5
- **Rejected**: Overkill for local use

## Implementation Notes

### Database Initialization

```bash
# scripts/init-claudarity-db.sh
sqlite3 ~/.claude/claudarity.db < schema.sql
```

### Query Patterns

```bash
# Fast full-text search
sqlite3 claudarity.db "
  SELECT * FROM feedback_entries 
  WHERE id IN (
    SELECT feedback_id FROM feedback_fts 
    WHERE feedback_fts MATCH 'authentication'
  )
  ORDER BY ts DESC LIMIT 10;
"
```

### Maintenance

```bash
# Periodic optimization
sqlite3 claudarity.db "VACUUM;"
sqlite3 claudarity.db "ANALYZE;"
```

### Backup

```bash
# Simple file copy
cp ~/.claude/claudarity.db ~/.claude/backups/claudarity-$(date +%Y%m%d).db
```

## Performance Characteristics

- **Insert**: ~1ms per record (background)
- **Query**: <10ms for most queries
- **FTS Search**: <50ms for full-text search
- **Database Size**: ~1MB per 1000 feedback entries
- **Concurrent Readers**: Unlimited
- **Concurrent Writers**: Serialized (rarely a bottleneck)

## Security Considerations

- **File Permissions**: 600 (user-only access)
- **SQL Injection**: All queries use parameterized statements
- **Sensitive Data**: No credentials stored (use system keychain)
- **Encryption**: OS-level encryption (FileVault, LUKS)

## Future Enhancements

- **WAL Mode**: Enable Write-Ahead Logging for better concurrency
- **Compression**: Compress old entries to save space
- **Partitioning**: Separate tables by time period
- **Replication**: Optional sync to remote backup
- **Encryption**: SQLite encryption extension (SQLCipher)

## References

- SQLite Documentation: https://www.sqlite.org/docs.html
- FTS5 Documentation: https://www.sqlite.org/fts5.html
- `scripts/init-claudarity-db.sh`: Database initialization
- `hooks/log-feedback.sh`: Example write operations
- `scripts/auto-context-recall.sh`: Example read operations

## Review Date

Review after 1 year of production use to assess:
- Performance at scale (100k+ records)
- Concurrent access patterns
- Need for more advanced features
- Backup/restore effectiveness
