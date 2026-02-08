---
name: Bug Report
about: Report a bug or issue with Claudarity
title: '[BUG] '
labels: bug
assignees: ''
---

## Bug Description

A clear and concise description of what the bug is.

## To Reproduce

Steps to reproduce the behavior:
1. Go to '...'
2. Run command '...'
3. See error

## Expected Behavior

A clear and concise description of what you expected to happen.

## Actual Behavior

What actually happened instead.

## Environment

- **OS**: [e.g., macOS 13.0, Ubuntu 22.04]
- **Shell**: [e.g., bash 5.1, zsh 5.8]
- **Claude Code Version**: [e.g., 1.2.3]
- **Claudarity Version**: [e.g., 2.0.0]
- **Python Version**: [e.g., 3.9.0] (if relevant)

## Logs

Please include relevant log files:

```
# From ~/.claude/logs/error.log
[paste error logs here]
```

```
# From ~/.claude/logs/debug.log
[paste debug logs here]
```

## Database State

If relevant, include database information:

```bash
sqlite3 ~/.claude/claudarity.db "SELECT COUNT(*) FROM feedback_entries;"
# Output: [paste here]
```

## Configuration

If relevant, include your configuration (remove sensitive data):

```json
{
  "relevant": "config here"
}
```

## Screenshots

If applicable, add screenshots to help explain your problem.

## Additional Context

Add any other context about the problem here.

## Possible Solution

If you have ideas on how to fix this, please share them here.

## Related Issues

Link to any related issues: #123
