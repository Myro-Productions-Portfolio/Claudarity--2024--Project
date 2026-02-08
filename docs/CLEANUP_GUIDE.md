# Claudarity Cleanup & Modernization Guide

This guide documents the cleanup process for Claudarity V2.0 and provides a roadmap for modernizing the system to current standards.

## Overview

Claudarity V2.0 was an early experiment in reinforcement learning-based memory for Claude Code. It implemented:
- Win/loss feedback tracking
- Pattern-based feedback detection
- Template evolution based on adoption rates
- Confidence scoring for responses
- SQLite + Markdown dual storage

This guide covers:
1. What was built and why
2. What worked well
3. What needs improvement
4. How to modernize for current standards

## What Was Built

### Core Components

#### 1. Feedback Learning System
**Purpose**: Learn from user praise/criticism to improve responses

**Implementation**:
- Pattern-based detection (phrases/words)
- Short message filtering (≤5 words, ≤50 chars)
- Win/loss classification
- SQLite + Markdown storage

**Files**:
- `hooks/log-feedback.sh` - Main detection logic
- `config/feedback-patterns.template.json` - Pattern definitions
- `scripts/confidence-calculator.sh` - Confidence scoring

**Status**: ✅ Working, needs refinement

#### 2. Template Evolution System
**Purpose**: Evolve project templates based on real-world usage

**Implementation**:
- Track template applications
- Monitor user modifications
- Calculate adoption rates
- Generate evolution proposals

**Files**:
- `scripts/template-evolver.py` - Evolution analysis
- `commands/review-templates.md` - User review interface

**Status**: ⚠️ Partially implemented, needs completion

#### 3. Context Memory System
**Purpose**: Store and recall conversation history

**Implementation**:
- SQLite with FTS5 full-text search
- Session-based organization
- Project-specific context

**Files**:
- `scripts/auto-context-recall.sh` - Context retrieval
- `commands/gomemory.md` - User interface

**Status**: ✅ Working, needs optimization

#### 4. Event Hook System
**Purpose**: React to Claude Code lifecycle events

**Implementation**:
- Bash hooks for events
- Background processing (non-blocking)
- JSON input/output

**Files**:
- `hooks/*.sh` - Event handlers
- `settings.template.json` - Hook registration

**Status**: ✅ Working well

## What Worked Well

### ✅ Strengths

1. **Non-Blocking Architecture**
   - Background processing doesn't slow user interaction
   - Hooks fork immediately and return

2. **Dual Storage Strategy**
   - SQLite for fast queries
   - Markdown for human readability
   - Good balance of performance and transparency

3. **Configurable Patterns**
   - Easy to add new feedback patterns
   - No code changes needed
   - User-customizable

4. **Event-Driven Design**
   - Clean separation of concerns
   - Easy to add new hooks
   - Composable

5. **Local-First Privacy**
   - All data stays on user's machine
   - No cloud dependencies
   - Full user control

## What Needs Improvement

### ⚠️ Issues & Limitations

#### 1. Pattern Detection Accuracy
**Problem**: Simple keyword matching misses nuanced feedback

**Examples**:
- Sarcasm: "oh great, another bug" (detected as praise)
- Negation: "not bad" (detected as criticism)
- Context: "this is terrible... terribly good!" (ambiguous)

**Solution**: Add ML-based sentiment analysis as fallback

#### 2. Cold Start Problem
**Problem**: New templates/patterns have no confidence data

**Impact**: System can't make good recommendations initially

**Solution**: 
- Better default confidence scores
- Faster learning with smaller sample sizes
- Transfer learning from similar templates

#### 3. Hardcoded Paths
**Problem**: Many scripts have hardcoded paths like `/Volumes/DevDrive/Cache`

**Impact**: Not portable across systems

**Solution**: Use environment variables and config files

#### 4. Limited Error Handling
**Problem**: Background processes can fail silently

**Impact**: Hard to debug issues

**Solution**: Better logging, monitoring, and error reporting

#### 5. No Automated Testing
**Problem**: No test suite for hooks/scripts

**Impact**: Regressions hard to catch

**Solution**: Add integration tests

#### 6. Template Evolution Incomplete
**Problem**: Evolution proposals generated but not applied

**Impact**: Templates don't actually evolve

**Solution**: Complete the approval and application workflow

## Cleanup Tasks

### Phase 1: Documentation (DONE ✅)

- [x] Create ADRs for key decisions
- [x] Document architecture and design
- [x] Explain reinforcement learning approach
- [x] Create this cleanup guide

### Phase 2: Code Cleanup

#### 2.1 Remove Hardcoded Paths

**Files to update**:
```bash
hooks/log-feedback.sh
scripts/confidence-calculator.sh
scripts/template-evolver.py
```

**Changes**:
```bash
# Before
CACHE_DIR="/Volumes/DevDrive/Cache/feedback"

# After
CACHE_DIR="${CLAUDARITY_CACHE_DIR:-$HOME/.claude/cache/feedback}"
```

#### 2.2 Improve Error Handling

**Pattern to apply**:
```bash
# Add to all hooks
set -euo pipefail
trap 'echo "Error on line $LINENO" >> "$HOME/.claude/logs/error.log"' ERR

# Validate inputs
if [ -z "$input" ]; then
  echo "$(date): ERROR - No input received" >> "$HOME/.claude/logs/error.log"
  exit 1
fi
```

#### 2.3 Add Configuration File

**Create**: `config/claudarity.conf`
```bash
# Claudarity Configuration

# Storage locations
CLAUDARITY_DB="${HOME}/.claude/claudarity.db"
CLAUDARITY_CACHE_DIR="${HOME}/.claude/cache"
CLAUDARITY_LOG_DIR="${HOME}/.claude/logs"

# Thresholds
ADOPTION_THRESHOLD=0.70
WIN_RATE_THRESHOLD=0.75
CONFIDENCE_THRESHOLD=0.70
MIN_SAMPLE_SIZE=10

# Feedback detection
MAX_FEEDBACK_WORDS=5
MAX_FEEDBACK_CHARS=50

# Maintenance
LOG_ROTATION_DAYS=30
DB_VACUUM_DAYS=7
```

#### 2.4 Standardize Logging

**Create**: `scripts/lib/logging.sh`
```bash
#!/usr/bin/env bash
# Logging utilities

LOG_DIR="${CLAUDARITY_LOG_DIR:-$HOME/.claude/logs}"

log_info() {
  echo "$(date '+%Y-%m-%d %H:%M:%S') [INFO] $*" >> "$LOG_DIR/info.log"
}

log_error() {
  echo "$(date '+%Y-%m-%d %H:%M:%S') [ERROR] $*" >> "$LOG_DIR/error.log"
}

log_debug() {
  if [ "${CLAUDARITY_DEBUG:-0}" = "1" ]; then
    echo "$(date '+%Y-%m-%d %H:%M:%S') [DEBUG] $*" >> "$LOG_DIR/debug.log"
  fi
}
```

### Phase 3: Testing

#### 3.1 Create Test Framework

**Create**: `tests/test-framework.sh`
```bash
#!/usr/bin/env bash
# Simple test framework

TESTS_PASSED=0
TESTS_FAILED=0

assert_equals() {
  local expected="$1"
  local actual="$2"
  local message="${3:-Assertion failed}"
  
  if [ "$expected" = "$actual" ]; then
    echo "✓ $message"
    ((TESTS_PASSED++))
  else
    echo "✗ $message"
    echo "  Expected: $expected"
    echo "  Actual: $actual"
    ((TESTS_FAILED++))
  fi
}

run_tests() {
  echo "Running tests..."
  echo "Passed: $TESTS_PASSED"
  echo "Failed: $TESTS_FAILED"
  [ $TESTS_FAILED -eq 0 ]
}
```

#### 3.2 Add Hook Tests

**Create**: `tests/test-log-feedback.sh`
```bash
#!/usr/bin/env bash
source tests/test-framework.sh

test_praise_detection() {
  result=$(echo '{"prompt":"awesome!","session_id":"test"}' | hooks/log-feedback.sh)
  # Check if logged as win
  assert_equals "win" "$result" "Detects praise"
}

test_criticism_detection() {
  result=$(echo '{"prompt":"terrible","session_id":"test"}' | hooks/log-feedback.sh)
  # Check if logged as loss
  assert_equals "loss" "$result" "Detects criticism"
}

test_long_message_filtered() {
  result=$(echo '{"prompt":"this is a very long message that should be filtered out","session_id":"test"}' | hooks/log-feedback.sh)
  # Check if not logged
  assert_equals "" "$result" "Filters long messages"
}

run_tests
```

### Phase 4: Modernization

#### 4.1 Add ML-Based Sentiment Analysis

**Option 1: Local Model (Recommended)**
```python
# scripts/sentiment-analyzer.py
from transformers import pipeline

classifier = pipeline("sentiment-analysis", 
                     model="distilbert-base-uncased-finetuned-sst-2-english")

def analyze_sentiment(text):
    result = classifier(text)[0]
    return {
        'label': result['label'],  # POSITIVE or NEGATIVE
        'confidence': result['score']
    }
```

**Option 2: Claude API (Fallback)**
```bash
# Use Claude API for nuanced sentiment analysis
analyze_with_claude() {
  local message="$1"
  curl -s https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -d "{\"model\":\"claude-3-haiku-20240307\",\"messages\":[{\"role\":\"user\",\"content\":\"Is this feedback positive or negative? Reply with just POSITIVE or NEGATIVE: $message\"}]}"
}
```

#### 4.2 Implement Template Evolution Workflow

**Complete the cycle**:
1. ✅ Generate proposals (done)
2. ⚠️ User review interface (partial)
3. ❌ Apply approved changes (missing)
4. ❌ Version management (missing)
5. ❌ Rollback mechanism (missing)

**Create**: `scripts/apply-template-evolution.sh`
```bash
#!/usr/bin/env bash
# Apply approved template evolution proposals

proposal_id="$1"
proposal_file="/path/to/proposals.json"

# Read proposal
proposal=$(jq ".[] | select(.proposal_id == \"$proposal_id\")" "$proposal_file")

# Apply changes
template_id=$(echo "$proposal" | jq -r '.template_id')
changes=$(echo "$proposal" | jq -r '.changes[]')

# For each change, add to template
# Update version
# Create changelog
# Test template
```

#### 4.3 Add Monitoring Dashboard

**Create**: `scripts/dashboard.sh`
```bash
#!/usr/bin/env bash
# Simple terminal dashboard

echo "=== Claudarity Dashboard ==="
echo ""
echo "Feedback Stats:"
sqlite3 ~/.claude/claudarity.db "
  SELECT 
    type,
    COUNT(*) as count,
    ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM feedback_entries), 2) as percentage
  FROM feedback_entries
  GROUP BY type;
"

echo ""
echo "Top Templates:"
sqlite3 ~/.claude/claudarity.db "
  SELECT template_id, confidence_score, application_count
  FROM template_confidence
  ORDER BY confidence_score DESC
  LIMIT 10;
"

echo ""
echo "Recent Activity:"
sqlite3 ~/.claude/claudarity.db "
  SELECT ts, type, pattern, project
  FROM feedback_entries
  ORDER BY ts DESC
  LIMIT 10;
"
```

## Migration to Current Standards

### Modern AI IDE Features to Consider

#### 1. MCP Servers
**Add**: MCP server for Claudarity

**Benefits**:
- Standardized tool interface
- Easier to extend

**Create**: `mcp-server/claudarity-mcp-server.py`
```python
from mcp.server import Server
from mcp.types import Tool, TextContent

server = Server("claudarity")

@server.list_tools()
async def list_tools():
    return [
        Tool(
            name="search_feedback",
            description="Search feedback history",
            inputSchema={
                "type": "object",
                "properties": {
                    "query": {"type": "string"},
                    "type": {"type": "string", "enum": ["win", "loss", "all"]}
                }
            }
        ),
        Tool(
            name="get_baseline_stats",
            description="Get win/loss statistics",
            inputSchema={"type": "object", "properties": {}}
        )
    ]

@server.call_tool()
async def call_tool(name, arguments):
    if name == "search_feedback":
        # Query SQLite
        results = search_feedback_db(arguments["query"], arguments.get("type"))
        return [TextContent(type="text", text=format_results(results))]
    elif name == "get_baseline_stats":
        stats = get_baseline_stats_db()
        return [TextContent(type="text", text=format_stats(stats))]
```

#### 2. Steering / Preference Files
**Add**: Configuration files for learned preferences

**Benefits**:
- Automatic context injection
- Easier to manage

Store learned preferences in a structured file (e.g., `config/learned-preferences.md`) that can be injected into AI assistant context automatically.

## Recommended Next Steps

### Immediate (Week 1)
1. ✅ Complete documentation (ADRs, guides)
2. Remove hardcoded paths
3. Add configuration file
4. Improve error handling

### Short-term (Month 1)
1. Add test framework
2. Write tests for core hooks
3. Standardize logging
4. Create monitoring dashboard

### Medium-term (Quarter 1)
1. Add ML-based sentiment analysis
2. Complete template evolution workflow
3. Create MCP server

### Long-term (Year 1)
1. Add cross-device sync
2. Implement team collaboration features
3. Build web dashboard
4. Add advanced analytics

## Success Metrics

### Current Baseline
- Feedback detection accuracy: ~77% F1 score
- Template evolution: Not measured (incomplete)
- User satisfaction: Not measured

### Target Metrics
- Feedback detection accuracy: >90% F1 score
- Template evolution adoption: >50% of proposals accepted
- User satisfaction: >4/5 rating
- System reliability: >99% uptime

## Conclusion

Claudarity V2.0 was a successful experiment in reinforcement learning for code assistants. The core ideas are sound:
- Learning from feedback
- Evolving templates
- Building confidence over time

With cleanup and modernization, this can become a production-ready system that genuinely improves over time based on user behavior.

The key is to:
1. Fix the rough edges (hardcoded paths, error handling)
2. Complete the incomplete features (template evolution)
3. Add modern capabilities (ML sentiment, MCP servers)

This guide provides the roadmap. The foundation is solid—now it's time to build on it.
