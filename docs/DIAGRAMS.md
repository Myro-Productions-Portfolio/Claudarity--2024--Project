# Claudarity System Diagrams

Visual representations of Claudarity's architecture and workflows.

## System Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                         USER INTERACTION                         │
│                      (Claude Code CLI)                             │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                         EVENT HOOKS                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │
│  │PromptSubmit  │  │ SessionStart │  │     Stop     │         │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘         │
└─────────┼──────────────────┼──────────────────┼─────────────────┘
          │                  │                  │
          ▼                  ▼                  ▼
┌─────────────────────────────────────────────────────────────────┐
│                      PROCESSING LAYER                            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │
│  │   Feedback   │  │   Context    │  │  Template    │         │
│  │  Detection   │  │   Recall     │  │  Evolution   │         │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘         │
└─────────┼──────────────────┼──────────────────┼─────────────────┘
          │                  │                  │
          ▼                  ▼                  ▼
┌─────────────────────────────────────────────────────────────────┐
│                       STORAGE LAYER                              │
│  ┌──────────────────────────────┐  ┌──────────────────────┐    │
│  │      SQLite Database         │  │   Markdown Cache     │    │
│  │  - feedback_entries          │  │  - win-*.md          │    │
│  │  - feedback_fts (search)     │  │  - loss-*.md         │    │
│  │  - template_confidence       │  │  - Human readable    │    │
│  │  - context_memory            │  │  - Easy backup       │    │
│  └──────────────────────────────┘  └──────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
```

## Reinforcement Learning Loop

```
┌─────────────────────────────────────────────────────────────────┐
│                    1. INITIAL STATE                              │
│  • User asks for help                                           │
│  • System has confidence scores                                 │
│  • Historical context available                                 │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│                    2. ACTION SELECTION                           │
│  • Choose template (ε-greedy: 90% exploit, 10% explore)        │
│  • Select patterns to apply                                     │
│  • Decide context to inject                                     │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│                    3. EXECUTE ACTION                             │
│  • Claude generates response                                    │
│  • Apply selected template/patterns                             │
│  • Include relevant context                                     │
│  • Log action taken                                             │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│                    4. OBSERVE REWARD                             │
│  • User provides feedback (or not)                              │
│  • Detect praise/criticism patterns                             │
│  • Classify: win (+1), loss (-1), neutral (0)                  │
│  • Log outcome with context                                     │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│                    5. UPDATE BELIEFS                             │
│  • Update confidence scores                                     │
│  • Adjust template preferences                                  │
│  • Update user preference profile                               │
│  • Log for future analysis                                      │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│                    6. PERIODIC LEARNING                          │
│  • Analyze patterns (weekly)                                    │
│  • Generate evolution proposals                                 │
│  • Update templates (if approved)                               │
│  • Refine confidence formulas                                   │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         └──────────► Back to step 1
```

## Feedback Detection Flow

```
User Message: "awesome work!"
         │
         ▼
┌─────────────────────────────────────┐
│   1. Length Filter                  │
│   • Word count: 2 ≤ 5 ✓            │
│   • Char count: 13 ≤ 50 ✓          │
│   • Passes filter                   │
└─────────────┬───────────────────────┘
              │
              ▼
┌─────────────────────────────────────┐
│   2. Normalize                      │
│   • Convert to lowercase            │
│   • "awesome work!"                 │
└─────────────┬───────────────────────┘
              │
              ▼
┌─────────────────────────────────────┐
│   3. Check Phrases                  │
│   • "great job" ✗                   │
│   • "well done" ✗                   │
│   • "awesome work" ✗                │
│   • No phrase match                 │
└─────────────┬───────────────────────┘
              │
              ▼
┌─────────────────────────────────────┐
│   4. Check Words                    │
│   • "awesome" ✓ MATCH!              │
│   • Pattern: "awesome"              │
└─────────────┬───────────────────────┘
              │
              ▼
┌─────────────────────────────────────┐
│   5. Classify                       │
│   • Type: WIN                       │
│   • Pattern: "awesome"              │
│   • Confidence: High                │
└─────────────┬───────────────────────┘
              │
              ▼
┌─────────────────────────────────────┐
│   6. Extract Context                │
│   • Recent files: auth.ts, api.ts   │
│   • Recent actions: implemented JWT │
│   • Project: my-app                 │
└─────────────┬───────────────────────┘
              │
              ▼
┌─────────────────────────────────────┐
│   7. Generate AI Summary            │
│   • Call Claude API (optional)      │
│   • Summarize conversation          │
│   • Extract key points              │
└─────────────┬───────────────────────┘
              │
              ▼
┌─────────────────────────────────────┐
│   8. Store Feedback                 │
│   • SQLite: INSERT INTO feedback    │
│   • Markdown: win-2024-01-15.md     │
│   • Update confidence scores        │
└─────────────────────────────────────┘
```

## Template Evolution Workflow

```
Template Applied
      │
      ▼
┌─────────────────────────────────────┐
│   Track Usage                       │
│   • Log template_id                 │
│   • Log project_path                │
│   • Create metadata file            │
└─────────────┬───────────────────────┘
              │
              ▼
┌─────────────────────────────────────┐
│   Monitor Modifications             │
│   • User adds src/hooks/            │
│   • User adds src/utils/api.ts      │
│   • Log all additions               │
└─────────────┬───────────────────────┘
              │
              ▼
┌─────────────────────────────────────┐
│   Collect Feedback                  │
│   • User: "great job!"              │
│   • Associate with template         │
│   • Log outcome: WIN                │
└─────────────┬───────────────────────┘
              │
              ▼
┌─────────────────────────────────────┐
│   Periodic Analysis (Weekly)        │
│   • Aggregate modifications         │
│   • Calculate adoption rates        │
│   • Calculate win rates             │
│   • Calculate confidence scores     │
└─────────────┬───────────────────────┘
              │
              ▼
┌─────────────────────────────────────┐
│   Check Thresholds                  │
│   • Adoption ≥ 70%? ✓               │
│   • Win rate ≥ 75%? ✓               │
│   • Confidence ≥ 70%? ✓             │
│   • Sample size ≥ 10? ✓             │
└─────────────┬───────────────────────┘
              │
              ▼
┌─────────────────────────────────────┐
│   Generate Proposal                 │
│   • Proposal ID: PROP-0001          │
│   • Changes: Add src/hooks/         │
│   • Rationale: 85% adoption         │
│   • Status: Pending                 │
└─────────────┬───────────────────────┘
              │
              ▼
┌─────────────────────────────────────┐
│   User Review                       │
│   • /review-templates               │
│   • Show proposal details           │
│   • User approves/rejects           │
└─────────────┬───────────────────────┘
              │
              ▼
┌─────────────────────────────────────┐
│   Apply Changes (if approved)       │
│   • Update template files           │
│   • Increment version (1.0 → 1.1)   │
│   • Create changelog                │
│   • Test new template               │
└─────────────────────────────────────┘
```

## Confidence Score Calculation

```
Template: react-typescript-app
      │
      ▼
┌─────────────────────────────────────┐
│   Collect Data                      │
│   • Usage count: 15                 │
│   • Wins: 12                        │
│   • Losses: 3                       │
│   • Recent wins (90d): 8            │
│   • Recent total (90d): 10          │
│   • Avg project age: 60 days        │
└─────────────┬───────────────────────┘
              │
              ▼
┌─────────────────────────────────────┐
│   Calculate Factors                 │
│                                     │
│   Usage Factor (30%):               │
│   min(15/10, 1.0) = 1.0             │
│                                     │
│   Win Rate (40%):                   │
│   12/(12+3) = 0.80                  │
│                                     │
│   Recency Factor (10%):             │
│   8/10 = 0.80                       │
│                                     │
│   Maturity Factor (20%):            │
│   min(60/90, 1.0) = 0.67            │
└─────────────┬───────────────────────┘
              │
              ▼
┌─────────────────────────────────────┐
│   Calculate Confidence              │
│                                     │
│   confidence =                      │
│     1.0  * 0.3 +  (usage)           │
│     0.80 * 0.4 +  (win rate)        │
│     0.80 * 0.1 +  (recency)         │
│     0.67 * 0.2    (maturity)        │
│                                     │
│   = 0.30 + 0.32 + 0.08 + 0.13       │
│   = 0.83                            │
└─────────────┬───────────────────────┘
              │
              ▼
┌─────────────────────────────────────┐
│   Interpret Score                   │
│   • 0.83 = High Confidence          │
│   • Recommendation: Use this        │
│   • Show to user with confidence    │
└─────────────────────────────────────┘
```

## Hook Execution Flow

```
User submits prompt
      │
      ▼
┌─────────────────────────────────────┐
│   Claude Code Event                 │
│   • Event: UserPromptSubmit         │
│   • Data: {prompt, session_id, ...} │
└─────────────┬───────────────────────┘
              │
              ▼
┌─────────────────────────────────────┐
│   Hook Triggered                    │
│   • Load: log-feedback.sh           │
│   • Input: JSON on stdin            │
└─────────────┬───────────────────────┘
              │
              ▼
┌─────────────────────────────────────┐
│   Read Input (Synchronous)          │
│   input=$(cat)                      │
│   • MUST happen before fork         │
│   • stdin not available in bg       │
└─────────────┬───────────────────────┘
              │
              ▼
┌─────────────────────────────────────┐
│   Optional: Synchronous Work        │
│   • Check callbacks                 │
│   • Show immediate response         │
│   • Critical operations only        │
└─────────────┬───────────────────────┘
              │
              ▼
┌─────────────────────────────────────┐
│   Fork to Background                │
│   (                                 │
│     # Background work               │
│   ) &                               │
│   • Non-blocking                    │
│   • Parallel processing             │
└─────────────┬───────────────────────┘
              │
              ├──────────────────────────┐
              │                          │
              ▼                          ▼
┌─────────────────────────┐  ┌─────────────────────────┐
│   Main Thread           │  │   Background Thread     │
│   • Return immediately  │  │   • Extract data        │
│   • User continues      │  │   • Process feedback    │
│   • No blocking         │  │   • Update database     │
│   exit 0                │  │   • Generate summary    │
└─────────────────────────┘  │   • Log completion      │
                             └─────────────────────────┘
```

## Data Flow: Feedback to Learning

```
User: "great job!"
      │
      ▼
┌─────────────────────────────────────┐
│   Feedback Detection                │
│   • Pattern match: "great job"      │
│   • Classification: WIN             │
└─────────────┬───────────────────────┘
              │
              ▼
┌─────────────────────────────────────┐
│   Context Extraction                │
│   • Recent files: auth.ts           │
│   • Recent patterns: JWT tokens     │
│   • Project: my-app                 │
└─────────────┬───────────────────────┘
              │
              ▼
┌─────────────────────────────────────┐
│   Storage (Dual Write)              │
│   ┌─────────────┐  ┌──────────────┐│
│   │   SQLite    │  │   Markdown   ││
│   │   INSERT    │  │   CREATE     ││
│   │   feedback  │  │   win-*.md   ││
│   └─────────────┘  └──────────────┘│
└─────────────┬───────────────────────┘
              │
              ▼
┌─────────────────────────────────────┐
│   Immediate Learning                │
│   • Update confidence scores        │
│   • Increment win count             │
│   • Update recency factor           │
└─────────────┬───────────────────────┘
              │
              ▼
┌─────────────────────────────────────┐
│   Pattern Extraction                │
│   • Identify successful patterns    │
│   • "JWT authentication" → success  │
│   • Store in preferences            │
└─────────────┬───────────────────────┘
              │
              ▼
┌─────────────────────────────────────┐
│   Periodic Analysis (Weekly)        │
│   • Aggregate all feedback          │
│   • Identify trends                 │
│   • Generate evolution proposals    │
└─────────────┬───────────────────────┘
              │
              ▼
┌─────────────────────────────────────┐
│   Future Responses                  │
│   • Higher confidence in JWT        │
│   • Recommend similar patterns      │
│   • Apply learned preferences       │
└─────────────────────────────────────┘
```

## Database Schema Relationships

```
┌─────────────────────────────────────────────────────────────────┐
│                     feedback_entries                             │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ id (PK) | ts | project | pattern | type | user_message    │ │
│  │ context_summary | ai_summary | cache_file                  │ │
│  └────────────────────────────────────────────────────────────┘ │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         │ 1:1
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│                       feedback_fts                               │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ feedback_id (FK) | user_message | context_summary          │ │
│  │ ai_summary                                                  │ │
│  │ [Full-text search index]                                   │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                   template_confidence                            │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ template_id (PK) | confidence_score | application_count    │ │
│  │ win_rate | recency_factor | maturity_factor                │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                     context_memory                               │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ id (PK) | ts | session_id | role | content | project       │ │
│  │ files_mentioned | commands_used                            │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## File System Layout

```
~/.claude/
├── claudarity.db                    # SQLite database
├── config/
│   ├── feedback-patterns.json       # Feedback detection patterns
│   ├── settings.json                # System settings
│   └── claudarity.conf              # Configuration (to be added)
├── hooks/
│   ├── log-feedback.sh              # Feedback detection hook
│   ├── session-start.sh             # Session initialization
│   ├── backup-session-log.sh        # Session backup
│   └── ...
├── scripts/
│   ├── confidence-calculator.sh     # Confidence scoring
│   ├── template-evolver.py          # Template evolution
│   ├── auto-context-recall.sh       # Context retrieval
│   └── ...
├── commands/
│   ├── baseline.md                  # Win/loss stats command
│   ├── gomemory.md                  # Context search command
│   └── ...
└── logs/
    ├── debug.log                    # Debug logging
    ├── error.log                    # Error logging
    └── info.log                     # Info logging

/Volumes/DevDrive/Cache/             # External cache (to be made configurable)
└── feedback/
    ├── win-2024-01-15-10-30-00.md   # Win feedback (markdown)
    ├── loss-2024-01-15-11-00-00.md  # Loss feedback (markdown)
    └── ...
```

## Migration Phases

```
┌─────────────────────────────────────────────────────────────────┐
│                    PHASE 1: STABILIZATION                        │
│                         (4 weeks)                                │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Week 1: Code Cleanup                                       │ │
│  │  • Remove hardcoded paths                                  │ │
│  │  • Add configuration file                                  │ │
│  │  • Standardize logging                                     │ │
│  │  • Improve error handling                                  │ │
│  └────────────────────────────────────────────────────────────┘ │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Week 2: Testing Infrastructure                             │ │
│  │  • Create test framework                                   │ │
│  │  • Write unit tests                                        │ │
│  │  • Write integration tests                                 │ │
│  │  • Set up CI/CD                                            │ │
│  └────────────────────────────────────────────────────────────┘ │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Week 3: Documentation                                      │ │
│  │  • Update README                                           │ │
│  │  • Create user guide                                       │ │
│  │  • Create developer guide                                  │ │
│  │  • Document all ADRs                                       │ │
│  └────────────────────────────────────────────────────────────┘ │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Week 4: Bug Fixes & Polish                                 │ │
│  │  • Fix known issues                                        │ │
│  │  • Performance optimization                                │ │
│  │  • Security audit                                          │ │
│  │  • User testing                                            │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│                    PHASE 2: MODERNIZATION                        │
│                         (4 weeks)                                │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Week 5: ML Integration                                     │ │
│  │  • Add sentiment analysis                                  │ │
│  │  • Improve feedback detection                              │ │
│  │  • Benchmark accuracy                                      │ │
│  │  • Target: >90% F1 score                                   │ │
│  └────────────────────────────────────────────────────────────┘ │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Week 6: Template Evolution                                 │ │
│  │  • Complete evolution workflow                             │ │
│  │  • Add version management                                  │ │
│  │  • Add rollback mechanism                                  │ │
│  │  • Test evolved templates                                  │ │
│  └────────────────────────────────────────────────────────────┘ │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Week 7: Monitoring                                         │ │
│  │  • Create dashboard                                        │ │
│  │  • Add health checks                                       │ │
│  │  • Add alerting                                            │ │
│  │  • Performance monitoring                                  │ │
│  └────────────────────────────────────────────────────────────┘ │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Week 8: Advanced Features                                  │ │
│  │  • Confidence visualization                                │ │
│  │  • Learning insights                                       │ │
│  │  • Customization options                                   │ │
│  │  • Export/import settings                                  │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│                    PHASE 3: INTEGRATION                          │
│                         (4 weeks)                                │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Week 9: Plugin Architecture                                │ │
│  │  • Create plugin structure                                 │ │
│  │  • Implement MCP server                                    │ │
│  │  • Create configuration files                              │ │
│  │  • Test integration                                        │ │
│  └────────────────────────────────────────────────────────────┘ │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Week 10: Hook System                                       │ │
│  │  • Refine hook architecture                                │ │
│  │  • Add hook UI                                             │ │
│  │  • Improve reliability                                     │ │
│  │  • Test hook execution                                     │ │
│  └────────────────────────────────────────────────────────────┘ │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Week 11: Integration Testing                               │ │
│  │  • End-to-end testing                                      │ │
│  │  • Performance testing                                     │ │
│  │  • User acceptance testing                                 │ │
│  │  • Fix critical issues                                     │ │
│  └────────────────────────────────────────────────────────────┘ │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Week 12: Launch                                            │ │
│  │  • Final polish                                            │ │
│  │  • Release preparation                                     │ │
│  │  • Publish plugin package                                   │ │
│  │  • Monitor and support                                     │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

---

**Note**: These diagrams are text-based for portability. For presentations, consider creating visual diagrams using tools like Mermaid, PlantUML, or draw.io.
