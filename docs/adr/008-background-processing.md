# ADR-008: Background Processing Model

## Status

Accepted (2024)

## Context

Event hooks need to perform potentially time-consuming operations:
- Database writes
- File I/O
- API calls (for AI summaries)
- Log processing
- Pattern analysis

These operations must not block user interaction. Users expect:
- Instant response when submitting prompts
- No lag in Claude's responses
- Smooth, uninterrupted workflow

## Decision

Implement a **fork-to-background** processing model where:

1. **Read input synchronously** (before forking)
2. **Fork to background immediately** (non-blocking)
3. **Process in background** (parallel to user interaction)
4. **Return immediately** (main thread continues)

### Implementation Pattern

```bash
#!/usr/bin/env bash
# hook-name.sh

# CRITICAL: Read stdin BEFORE forking
# (stdin not available in background process)
input=$(cat)

# Optional: Synchronous pre-processing
# (only for operations that MUST complete before returning)
if [ "$CRITICAL_CHECK" = "true" ]; then
  # Do critical work here
fi

# Fork to background IMMEDIATELY
(
  # All heavy processing happens here
  # Extract data from input
  user_msg=$(echo "$input" | jq -r '.prompt')
  
  # Perform operations
  process_data "$user_msg"
  update_database
  generate_summary
  
  # Log completion
  echo "$(date): Processing complete" >> "$LOG_FILE"
  
) &  # & = run in background

# Return immediately (non-blocking)
exit 0
```

## Consequences

### Positive

- **Non-Blocking**: User interaction never delayed
- **Parallel Processing**: Multiple hooks can run simultaneously
- **Simple**: Easy to understand and implement
- **Portable**: Works on any Unix-like system
- **Scalable**: Can handle many concurrent operations
- **Resilient**: Background failures don't affect user

### Negative

- **Error Handling**: Background errors can fail silently
- **Debugging**: Harder to debug background processes
- **Resource Usage**: Many background processes can accumulate
- **No Feedback**: User doesn't see processing status
- **Race Conditions**: Concurrent access to shared resources
- **Orphaned Processes**: Background processes can outlive parent

### Mitigations

- **Logging**: All background operations log to files
- **Error Traps**: Catch and log errors in background
- **Resource Limits**: Rate limiting prevents accumulation
- **Timeouts**: Long-running processes have timeouts
- **Locking**: Database handles concurrent access
- **Cleanup**: Periodic cleanup of stale processes

## Alternatives Considered

### 1. Synchronous Processing
```bash
# Process everything before returning
input=$(cat)
process_data "$input"
update_database
exit 0
```

- **Pros**: Simple, no concurrency issues, immediate feedback
- **Cons**: Blocks user interaction, slow, poor UX
- **Rejected**: Unacceptable latency for users

### 2. Queue-Based Processing
```bash
# Add to queue, separate worker processes
input=$(cat)
echo "$input" >> "$QUEUE_FILE"
exit 0

# Separate worker
while true; do
  process_queue
  sleep 1
done
```

- **Pros**: Better resource management, retry logic, monitoring
- **Cons**: Complex, requires worker management, overkill
- **Rejected**: Too complex for initial version

### 3. Event Loop (Node.js style)
```javascript
// Async event processing
async function handleEvent(input) {
  await processData(input);
  await updateDatabase();
}
```

- **Pros**: Modern, good error handling, familiar pattern
- **Cons**: Requires Node.js, heavier runtime, more complex
- **Rejected**: Bash simpler for this use case

### 4. Systemd Services
```ini
# Systemd service for processing
[Service]
ExecStart=/path/to/processor
```

- **Pros**: Robust, automatic restart, logging
- **Cons**: Linux-only, complex setup, overkill
- **Rejected**: Not portable, too heavy

### 5. Cron Jobs
```bash
# Process periodically
*/5 * * * * /path/to/processor
```

- **Pros**: Simple, built-in scheduling
- **Cons**: Not event-driven, delayed processing, not real-time
- **Rejected**: Need immediate processing, not periodic

## Implementation Details

### Reading Input

```bash
# MUST read stdin before forking
# stdin not available in background process
input=$(cat)

# Can also read with timeout
input=$(timeout 5s cat) || {
  echo "$(date): ERROR - Timeout reading input" >> "$LOG_FILE"
  exit 1
}
```

### Forking to Background

```bash
# Parentheses create subshell
# & runs in background
(
  # Background work here
  echo "Processing..."
  sleep 5
  echo "Done"
) &

# Main script continues immediately
echo "Returned to user"
```

### Error Handling in Background

```bash
(
  # Set error handling
  set -euo pipefail
  
  # Trap errors
  trap 'echo "$(date): ERROR on line $LINENO" >> "$LOG_FILE"' ERR
  
  # Validate input
  if [ -z "$input" ]; then
    echo "$(date): ERROR - No input" >> "$LOG_FILE"
    exit 1
  fi
  
  # Process with error checking
  result=$(process_data "$input" 2>&1) || {
    echo "$(date): ERROR - Processing failed: $result" >> "$LOG_FILE"
    exit 1
  }
  
) &
```

### Resource Management

```bash
# Limit concurrent background processes
MAX_PROCESSES=10
current=$(pgrep -f "hook-name.sh" | wc -l)

if [ "$current" -gt "$MAX_PROCESSES" ]; then
  echo "$(date): WARN - Too many processes, skipping" >> "$LOG_FILE"
  exit 0
fi

# Fork with resource limits
(
  # Limit CPU time (10 seconds)
  ulimit -t 10
  
  # Limit memory (100MB)
  ulimit -v 102400
  
  # Process
  process_data
) &
```

### Timeouts

```bash
(
  # Run with timeout
  timeout 30s process_data || {
    echo "$(date): ERROR - Timeout after 30s" >> "$LOG_FILE"
    exit 1
  }
) &
```

## Performance Characteristics

### Latency

- **Hook startup**: <10ms
- **Fork overhead**: <5ms
- **Total user-facing latency**: <15ms
- **Background processing**: 100ms - 5s (doesn't affect user)

### Resource Usage

- **Memory per process**: ~2MB
- **CPU**: Minimal (mostly I/O bound)
- **Disk I/O**: Moderate (database writes, logs)
- **Max concurrent processes**: 10-20 (configurable)

### Scalability

- **Hooks per second**: 100+ (limited by system)
- **Concurrent hooks**: 10-20 (configurable)
- **Database contention**: Handled by SQLite locking
- **Log file contention**: Append-only, minimal contention

## Monitoring

### Process Monitoring

```bash
# Check running background processes
pgrep -f "hook-name.sh" | wc -l

# Check process age
ps -eo pid,etime,cmd | grep "hook-name.sh"

# Kill stale processes (>5 minutes)
ps -eo pid,etime,cmd | grep "hook-name.sh" | \
  awk '$2 ~ /^[0-9][0-9]:/ {print $1}' | xargs kill
```

### Log Monitoring

```bash
# Check for errors
tail -f ~/.claude/logs/error.log

# Count errors per hour
grep "ERROR" ~/.claude/logs/error.log | \
  awk '{print $1" "$2}' | cut -d: -f1 | uniq -c

# Alert on high error rate
error_count=$(grep "ERROR" ~/.claude/logs/error.log | wc -l)
if [ "$error_count" -gt 100 ]; then
  echo "High error rate: $error_count errors"
fi
```

### Performance Monitoring

```bash
# Measure hook latency
start=$(date +%s%N)
echo '{"prompt":"test"}' | ~/.claude/hooks/log-feedback.sh
end=$(date +%s%N)
latency=$(( (end - start) / 1000000 ))  # Convert to ms
echo "Latency: ${latency}ms"

# Should be <15ms
if [ "$latency" -gt 15 ]; then
  echo "WARNING: High latency"
fi
```

## Debugging

### Debug Mode

```bash
# Enable debug logging
export CLAUDARITY_DEBUG=1

# Run hook with debug output
echo '{"prompt":"test"}' | ~/.claude/hooks/log-feedback.sh

# Check debug log
tail -f ~/.claude/logs/debug.log
```

### Testing Background Processing

```bash
# Test hook in foreground (for debugging)
# Remove & to run synchronously
(
  # Background work here
  echo "Processing..."
  process_data
  echo "Done"
)  # No & = runs in foreground

# Or use wait to block until complete
(
  process_data
) &
wait  # Wait for background process to complete
```

### Tracing Execution

```bash
# Enable bash tracing
set -x

# Trace background process
(
  set -x
  process_data
) &> /tmp/trace.log &

# View trace
tail -f /tmp/trace.log
```

## Best Practices

### 1. Always Read Input First
```bash
# CORRECT
input=$(cat)
(
  process "$input"
) &

# WRONG - stdin not available in background
(
  input=$(cat)  # This will hang!
  process "$input"
) &
```

### 2. Use Error Traps
```bash
(
  trap 'echo "Error on line $LINENO" >> "$LOG_FILE"' ERR
  set -euo pipefail
  # Process
) &
```

### 3. Log Everything
```bash
(
  echo "$(date): Starting processing" >> "$LOG_FILE"
  process_data
  echo "$(date): Completed successfully" >> "$LOG_FILE"
) &
```

### 4. Validate Input
```bash
(
  if [ -z "$input" ]; then
    echo "$(date): ERROR - No input" >> "$LOG_FILE"
    exit 1
  fi
  # Process
) &
```

### 5. Use Timeouts
```bash
(
  timeout 30s process_data || {
    echo "$(date): ERROR - Timeout" >> "$LOG_FILE"
  }
) &
```

## Future Enhancements

### 1. Job Queue System
- Persistent queue for reliability
- Retry logic for failures
- Priority levels
- Job status tracking

### 2. Worker Pool
- Fixed number of worker processes
- Better resource management
- Load balancing
- Graceful shutdown

### 3. Progress Reporting
- Real-time progress updates
- User notifications
- Status dashboard
- Completion callbacks

### 4. Distributed Processing
- Process on multiple machines
- Load distribution
- Fault tolerance
- Scalability

## References

- Bash Background Jobs: https://www.gnu.org/software/bash/manual/html_node/Job-Control.html
- Process Management: https://www.gnu.org/software/bash/manual/html_node/Job-Control-Basics.html
- `hooks/log-feedback.sh`: Example implementation

## Review Date

Review after 6 months to assess:
- Error rates in background processing
- Resource usage patterns
- Need for more sophisticated job management
- User impact of background failures
