# ADR-003: Bash Hooks for Event-Driven Architecture

## Status

Accepted (2024)

## Context

The memory system needs to react to Claude Code lifecycle events:
- User submits a prompt
- Assistant generates a response
- Session starts/ends
- Files are modified
- Commands are executed

Requirements:
- **Non-blocking**: Hooks must not slow down user interaction
- **Reliable**: Must handle errors gracefully
- **Extensible**: Easy to add new hooks
- **Portable**: Works across macOS/Linux
- **Lightweight**: Minimal overhead

## Decision

Implement an **event-driven architecture using Bash hooks** that:

### Hook Types

1. **UserPromptSubmit**: Fires when user sends a message
   - Feedback detection
   - Context injection
   - Terminal activity capture

2. **Stop**: Fires when session ends or user stops
   - Session backup
   - Summary generation
   - Cleanup tasks

3. **SessionStart**: Fires when new session begins
   - Show feedback summary
   - Load preferences
   - Initialize context

4. **AssistantResponse**: Fires after Claude responds
   - Extract patterns
   - Update confidence scores
   - Log interactions

### Hook Architecture

```
Event Trigger (Claude Code)
    ↓
Hook Script Execution
    ↓
Fork to Background (non-blocking)
    ↓
Process Event Data
    ↓
Update Database/Files
    ↓
Return (main thread continues)
```

### Implementation Pattern

```bash
#!/usr/bin/env bash
# hook-name.sh

# Read input synchronously
input=$(cat)

# Fork to background immediately
(
  # Process in background
  # Extract data from input
  # Perform operations
  # Update storage
) &

# Return immediately (non-blocking)
exit 0
```

### Hook Registration

Hooks are registered in `settings.json`:

```json
{
  "hooks": {
    "UserPromptSubmit": [
      "~/.claude/hooks/log-feedback.sh",
      "~/.claude/hooks/context-detector.sh"
    ],
    "Stop": [
      "~/.claude/hooks/backup-session-log.sh"
    ]
  }
}
```

## Consequences

### Positive

- **Non-Blocking**: Background processing doesn't slow user interaction
- **Simple**: Bash scripts are easy to write and debug
- **Portable**: Works on any Unix-like system
- **Composable**: Multiple hooks can run for same event
- **Debuggable**: Easy to test hooks in isolation
- **No Dependencies**: Just bash and standard Unix tools
- **Flexible**: Can call Python, Ruby, or other languages from hooks

### Negative

- **Error Handling**: Background processes can fail silently
- **Debugging**: Harder to debug background processes
- **Ordering**: No guaranteed execution order between hooks
- **Resource Usage**: Many background processes can accumulate
- **Platform Specific**: Bash syntax varies slightly across systems
- **No Built-in Retry**: Failed hooks don't automatically retry

### Mitigations

- **Logging**: All hooks log to `~/.claude/logs/debug.log`
- **Error Codes**: Hooks return meaningful exit codes
- **Timeouts**: Long-running hooks have timeouts
- **Resource Limits**: Rate limiting prevents process accumulation
- **Testing**: Each hook has test script
- **Monitoring**: Health check scripts detect failures

## Alternatives Considered

### 1. Python Event System
- **Pros**: More structured, better error handling, richer libraries
- **Cons**: Slower startup, requires Python runtime, more complex
- **Rejected**: Bash is faster for simple event handling

### 2. Node.js Event Emitters
- **Pros**: Async by default, good ecosystem
- **Cons**: Requires Node.js, heavier runtime, overkill
- **Rejected**: Too heavy for simple hooks

### 3. Systemd Timers/Services
- **Pros**: Built-in scheduling, service management
- **Cons**: Linux-only, complex setup, not event-driven
- **Rejected**: Not portable, not event-driven

### 4. Cron Jobs
- **Pros**: Simple, built-in scheduling
- **Cons**: Time-based not event-based, not real-time
- **Rejected**: Need event-driven, not time-based

### 5. Message Queue (RabbitMQ, Redis)
- **Pros**: Reliable, scalable, retry logic
- **Cons**: Requires server, complex setup, overkill
- **Rejected**: Too heavy for local tool

## Implementation Notes

### Hook Input Format

Hooks receive JSON on stdin:

```json
{
  "prompt": "user message",
  "session_id": "abc123",
  "transcript_path": "/path/to/transcript.jsonl",
  "cwd": "/current/working/directory"
}
```

### Background Processing Pattern

```bash
# Read input BEFORE forking (stdin not available in background)
input=$(cat)

# Fork immediately
(
  # Extract data
  user_msg=$(echo "$input" | jq -r '.prompt')
  
  # Process
  # ...
  
  # Log errors
  echo "$(date): Processing complete" >> "$HOME/.claude/logs/debug.log"
) &

# Return immediately
exit 0
```

### Error Handling

```bash
set -euo pipefail  # Exit on error, undefined vars, pipe failures

# Trap errors
trap 'echo "Error on line $LINENO" >> "$HOME/.claude/logs/error.log"' ERR

# Validate input
if [ -z "$input" ]; then
  echo "$(date): ERROR - No input received" >> "$HOME/.claude/logs/error.log"
  exit 1
fi
```

### Testing Hooks

```bash
# Test hook with sample input
echo '{"prompt":"test","session_id":"test123"}' | ~/.claude/hooks/log-feedback.sh

# Check logs
tail -f ~/.claude/logs/debug.log
```

## Performance Characteristics

- **Hook Startup**: <10ms (bash script execution)
- **Fork Overhead**: <5ms (background process creation)
- **Total Latency**: <15ms (user doesn't notice)
- **Concurrent Hooks**: 10+ can run simultaneously
- **Memory**: ~2MB per background process
- **CPU**: Minimal (mostly I/O bound)

## Security Considerations

- **Input Validation**: All user input sanitized
- **Path Traversal**: File paths validated
- **Command Injection**: No eval or unquoted variables
- **Permissions**: Hooks run with user permissions only
- **Secrets**: No secrets in hook scripts (use env vars)

## Hook Development Guidelines

1. **Keep It Simple**: One responsibility per hook
2. **Fork Early**: Background processing ASAP
3. **Log Everything**: Debug logs for troubleshooting
4. **Handle Errors**: Graceful degradation
5. **Test Independently**: Each hook should work standalone
6. **Document Input**: Clear documentation of expected input
7. **Exit Codes**: Use meaningful exit codes
8. **Idempotent**: Safe to run multiple times

## Future Enhancements

- **Hook Priorities**: Control execution order
- **Conditional Hooks**: Run only if conditions met
- **Hook Chaining**: Output of one hook feeds another
- **Retry Logic**: Automatic retry on failure
- **Rate Limiting**: Prevent hook spam
- **Monitoring Dashboard**: Visual hook health status

## References

- `hooks/log-feedback.sh`: Example hook implementation
- `hooks/README.md`: Hook development guide
- Claude Code Hooks Documentation
- Bash Best Practices: https://google.github.io/styleguide/shellguide.html

## Review Date

Review after 6 months to assess:
- Hook reliability and error rates
- Performance impact on user experience
- Need for more sophisticated event system
- Hook development complexity
