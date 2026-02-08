# ADR-001: Reinforcement Learning Approach for Memory System

## Status

Accepted (2024)

## Context

Claude Code needed a memory system that could:
- Learn from user feedback over time
- Improve response quality based on past interactions
- Adapt to individual user preferences and coding styles
- Provide contextual awareness across sessions

Traditional approaches like simple logging or static templates don't adapt to user behavior. We needed a system that could learn what works and what doesn't through reinforcement.

## Decision

Implement a reinforcement learning-based memory system using a **win/loss feedback mechanism**:

### Core Principles

1. **Binary Feedback Classification**
   - **Wins (Praise)**: Positive feedback indicating successful responses
   - **Losses (Criticism)**: Negative feedback indicating unsuccessful responses

2. **Pattern-Based Detection**
   - Automatic detection of feedback through phrase/word matching
   - Configurable patterns in JSON for extensibility
   - Short message filtering (≤5 words, ≤50 chars) to capture genuine feedback

3. **Confidence Scoring**
   - Templates and patterns accumulate confidence scores based on outcomes
   - Higher confidence = more likely to be used/recommended
   - Scores decay over time to favor recent patterns

4. **Template Evolution**
   - Templates evolve based on adoption rates and win rates
   - Successful patterns get promoted to template defaults
   - Low-performing patterns get deprecated

### Reinforcement Loop

```
User Interaction
    ↓
Claude Response (using templates/patterns)
    ↓
User Feedback (praise/criticism)
    ↓
Pattern Detection & Logging
    ↓
Confidence Score Update
    ↓
Template Evolution (if thresholds met)
    ↓
Future Responses Influenced
```

## Consequences

### Positive

- **Adaptive Learning**: System improves over time based on actual user feedback
- **Personalization**: Learns individual user preferences and coding styles
- **Measurable**: Win/loss metrics provide clear performance indicators
- **Transparent**: Users can see baseline stats and understand what's being learned
- **Extensible**: Easy to add new feedback patterns or adjust thresholds

### Negative

- **Cold Start Problem**: New templates have no confidence data initially
- **Feedback Dependency**: System only learns when users provide explicit feedback
- **Pattern Limitations**: Relies on keyword matching which can miss nuanced feedback
- **Complexity**: More complex than simple logging systems
- **Storage Growth**: Accumulates data over time requiring maintenance

### Mitigations

- **Default Confidence**: New templates start with neutral 0.5 confidence
- **Implicit Signals**: Track template usage and project age as additional signals
- **Pattern Tuning**: Configurable patterns allow refinement over time
- **Cleanup Scripts**: Automated log rotation and database compaction
- **Thresholds**: Require minimum sample sizes before making evolution decisions

## Alternatives Considered

### 1. Simple Logging Only
- **Pros**: Simple, no learning complexity
- **Cons**: No adaptation, no improvement over time
- **Rejected**: Doesn't meet goal of learning system

### 2. ML-Based Sentiment Analysis
- **Pros**: More nuanced feedback detection
- **Cons**: Requires external dependencies, slower, more complex
- **Rejected**: Overkill for initial version, can add later

### 3. Explicit Rating System
- **Pros**: Clear, unambiguous feedback
- **Cons**: Interrupts workflow, users unlikely to rate consistently
- **Rejected**: Too intrusive for developer workflow

### 4. Implicit Signals Only (no explicit feedback)
- **Pros**: No user effort required
- **Cons**: Hard to distinguish success from failure
- **Rejected**: Too ambiguous, needs explicit feedback for learning

## Implementation Notes

- Feedback detection runs in background to avoid blocking user input
- SQLite provides ACID guarantees for concurrent access
- Python used for ML-heavy operations (template evolution)
- Bash used for event hooks and simple processing
- Confidence scores recalculated periodically, not on every interaction

## References

- `hooks/log-feedback.sh`: Main feedback detection implementation
- `scripts/confidence-calculator.sh`: Confidence score calculation
- `scripts/template-evolver.py`: Template evolution logic
- `config/feedback-patterns.template.json`: Feedback pattern definitions

## Review Date

This decision should be reviewed after 6 months of production use to assess:
- Accuracy of feedback detection
- Effectiveness of confidence scoring
- User satisfaction with learning behavior
- Need for more sophisticated ML approaches
