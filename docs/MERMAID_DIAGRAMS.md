# Claudarity Mermaid Diagrams

This document contains Mermaid diagrams that render in GitHub, GitLab, and many Markdown viewers.

## System Architecture Overview

```mermaid
graph TB
    subgraph "User Layer"
        User[User/Developer]
        CLI[Claude Code CLI]
    end
    
    subgraph "Event Layer"
        PromptHook[UserPromptSubmit Hook]
        StartHook[SessionStart Hook]
        StopHook[Stop Hook]
    end
    
    subgraph "Processing Layer"
        FeedbackDetect[Feedback Detection]
        ContextRecall[Context Recall]
        TemplateEvol[Template Evolution]
        ConfidenceCalc[Confidence Calculator]
    end
    
    subgraph "Storage Layer"
        SQLite[(SQLite Database)]
        Markdown[Markdown Cache]
    end
    
    User -->|Interacts| CLI
    CLI -->|Triggers| PromptHook
    CLI -->|Triggers| StartHook
    CLI -->|Triggers| StopHook
    
    PromptHook -->|Detects| FeedbackDetect
    PromptHook -->|Searches| ContextRecall
    StopHook -->|Analyzes| TemplateEvol
    
    FeedbackDetect -->|Updates| ConfidenceCalc
    TemplateEvol -->|Uses| ConfidenceCalc
    
    FeedbackDetect -->|Writes| SQLite
    FeedbackDetect -->|Writes| Markdown
    ContextRecall -->|Reads| SQLite
    TemplateEvol -->|Reads| SQLite
    ConfidenceCalc -->|Reads/Writes| SQLite
    
    style User fill:#e1f5ff
    style CLI fill:#e1f5ff
    style SQLite fill:#ffe1e1
    style Markdown fill:#ffe1e1
```

## Reinforcement Learning Loop

```mermaid
graph LR
    A[1. Initial State<br/>Current confidence scores<br/>Historical context] --> B[2. Action Selection<br/>Choose template<br/>ε-greedy: 90% exploit, 10% explore]
    B --> C[3. Execute Action<br/>Generate response<br/>Apply patterns<br/>Include context]
    C --> D[4. Observe Reward<br/>User feedback<br/>Win +1 / Loss -1 / Neutral 0]
    D --> E[5. Update Beliefs<br/>Update confidence<br/>Adjust preferences<br/>Log outcome]
    E --> F[6. Periodic Learning<br/>Weekly analysis<br/>Generate proposals<br/>Evolve templates]
    F --> A
    
    style A fill:#e3f2fd
    style B fill:#fff3e0
    style C fill:#f3e5f5
    style D fill:#e8f5e9
    style E fill:#fce4ec
    style F fill:#f1f8e9
```

## Feedback Detection Flow

```mermaid
flowchart TD
    Start([User Message: 'awesome work!']) --> LengthCheck{Length Check<br/>≤5 words?<br/>≤50 chars?}
    
    LengthCheck -->|No| Skip[Skip - Too Long]
    LengthCheck -->|Yes| Normalize[Normalize<br/>Convert to lowercase]
    
    Normalize --> CheckPhrases{Check Phrases<br/>'great job'<br/>'well done'<br/>etc.}
    
    CheckPhrases -->|Match| ClassifyWin[Classify as WIN]
    CheckPhrases -->|No Match| CheckWords{Check Words<br/>'awesome'<br/>'excellent'<br/>etc.}
    
    CheckWords -->|Match| ClassifyWin
    CheckWords -->|No Match| CheckLoss{Check Loss<br/>Patterns}
    
    CheckLoss -->|Match| ClassifyLoss[Classify as LOSS]
    CheckLoss -->|No Match| Neutral[No Classification]
    
    ClassifyWin --> ExtractContext[Extract Context<br/>Recent files<br/>Recent actions<br/>Project info]
    ClassifyLoss --> ExtractContext
    
    ExtractContext --> GenerateSummary[Generate AI Summary<br/>Optional: Call Claude API]
    
    GenerateSummary --> Store[Store Feedback<br/>SQLite + Markdown]
    
    Store --> UpdateConfidence[Update Confidence Scores]
    
    UpdateConfidence --> End([Complete])
    Skip --> End
    Neutral --> End
    
    style Start fill:#e3f2fd
    style ClassifyWin fill:#c8e6c9
    style ClassifyLoss fill:#ffcdd2
    style Store fill:#fff9c4
    style End fill:#e3f2fd
```

## Template Evolution Workflow

```mermaid
sequenceDiagram
    participant User
    participant Template
    participant Monitor
    participant Analyzer
    participant Proposal
    participant Review
    
    User->>Template: Apply template to project
    Template->>Monitor: Log usage
    
    User->>Template: Add src/hooks/ directory
    Template->>Monitor: Log modification
    
    User->>Template: Add src/utils/api.ts
    Template->>Monitor: Log modification
    
    User->>Monitor: Feedback: "great job!"
    Monitor->>Monitor: Associate with template
    
    Note over Analyzer: Weekly Analysis
    
    Analyzer->>Monitor: Aggregate modifications
    Analyzer->>Analyzer: Calculate adoption rate (85%)
    Analyzer->>Analyzer: Calculate win rate (82%)
    Analyzer->>Analyzer: Check thresholds
    
    alt Thresholds Met
        Analyzer->>Proposal: Generate proposal
        Proposal->>Review: PROP-0001: Add src/hooks/
        Review->>User: Show proposal
        User->>Review: Approve
        Review->>Template: Apply changes
        Template->>Template: Version 1.0 → 1.1
    else Thresholds Not Met
        Analyzer->>Analyzer: Wait for more data
    end
```

## Confidence Score Calculation

```mermaid
graph TD
    Start([Template: react-typescript-app]) --> CollectData[Collect Data<br/>Usage: 15<br/>Wins: 12<br/>Losses: 3<br/>Recent wins: 8/10<br/>Avg age: 60 days]
    
    CollectData --> CalcUsage[Usage Factor<br/>min15/10, 1.0 = 1.0<br/>Weight: 30%]
    CollectData --> CalcWinRate[Win Rate<br/>12/12+3 = 0.80<br/>Weight: 40%]
    CollectData --> CalcRecency[Recency Factor<br/>8/10 = 0.80<br/>Weight: 10%]
    CollectData --> CalcMaturity[Maturity Factor<br/>min60/90, 1.0 = 0.67<br/>Weight: 20%]
    
    CalcUsage --> Combine[Combine Factors<br/>1.0×0.3 + 0.80×0.4 +<br/>0.80×0.1 + 0.67×0.2]
    CalcWinRate --> Combine
    CalcRecency --> Combine
    CalcMaturity --> Combine
    
    Combine --> Result[Confidence Score<br/>0.83]
    
    Result --> Interpret{Interpret}
    Interpret -->|0.7-0.9| HighConf[High Confidence<br/>Recommend this template]
    
    style Start fill:#e3f2fd
    style Result fill:#c8e6c9
    style HighConf fill:#a5d6a7
```

## Hook Execution Pattern

```mermaid
sequenceDiagram
    participant User
    participant Claude as Claude Code
    participant Hook as Hook Script
    participant BG as Background Process
    participant DB as Database
    
    User->>Claude: Submit prompt
    Claude->>Hook: Trigger UserPromptSubmit
    
    Note over Hook: Read stdin (synchronous)
    Hook->>Hook: input=$(cat)
    
    Note over Hook: Optional sync work
    Hook->>User: Show callback response
    
    Note over Hook: Fork to background
    Hook->>BG: ( background work ) &
    Hook->>Claude: Return immediately (exit 0)
    
    Note over Claude,User: User continues working<br/>(no blocking)
    
    par Background Processing
        BG->>BG: Extract data from input
        BG->>BG: Process feedback
        BG->>DB: Write to SQLite
        BG->>BG: Generate summary
        BG->>BG: Log completion
    end
    
    Note over BG: Process completes<br/>(user unaware)
```

## Data Flow: Feedback to Learning

```mermaid
graph LR
    A[User: 'great job!'] --> B[Feedback Detection<br/>Pattern match: WIN]
    B --> C[Context Extraction<br/>Files: auth.ts<br/>Patterns: JWT]
    C --> D[Dual Storage]
    
    D --> E[SQLite<br/>INSERT feedback]
    D --> F[Markdown<br/>win-*.md]
    
    E --> G[Immediate Learning<br/>Update confidence<br/>Increment win count]
    F --> G
    
    G --> H[Pattern Extraction<br/>JWT auth → success]
    H --> I[Store Preferences]
    
    I --> J[Weekly Analysis<br/>Aggregate feedback<br/>Identify trends]
    J --> K[Evolution Proposals<br/>Template improvements]
    
    K --> L[Future Responses<br/>Higher confidence<br/>Better recommendations]
    
    style A fill:#e3f2fd
    style B fill:#fff3e0
    style G fill:#c8e6c9
    style L fill:#a5d6a7
```

## Database Schema Relationships

```mermaid
erDiagram
    FEEDBACK_ENTRIES ||--o| FEEDBACK_FTS : "indexed by"
    FEEDBACK_ENTRIES {
        int id PK
        text ts
        text project
        text pattern
        text type
        text user_message
        text context_summary
        text ai_summary
        text cache_file
    }
    
    FEEDBACK_FTS {
        int feedback_id FK
        text user_message
        text context_summary
        text ai_summary
    }
    
    TEMPLATE_CONFIDENCE {
        text template_id PK
        real confidence_score
        int application_count
        real win_rate
        real recency_factor
        real maturity_factor
        text last_calculated
    }
    
    CONTEXT_MEMORY {
        int id PK
        text ts
        text session_id
        text role
        text content
        text project
        text files_mentioned
        text commands_used
    }
    
    SESSION_LOG {
        int id PK
        text session_id UK
        text start_time
        text end_time
        text project_path
        int total_interactions
        int praise_count
        int loss_count
    }
```

## Migration Phases Timeline

```mermaid
gantt
    title Claudarity Migration Timeline (12 Weeks)
    dateFormat YYYY-MM-DD
    section Phase 1: Stabilization
    Code Cleanup           :p1w1, 2026-02-09, 7d
    Testing Infrastructure :p1w2, 2026-02-16, 7d
    Documentation         :p1w3, 2026-02-23, 7d
    Bug Fixes & Polish    :p1w4, 2026-03-02, 7d
    
    section Phase 2: Modernization
    ML Integration        :p2w1, 2026-03-09, 7d
    Template Evolution    :p2w2, 2026-03-16, 7d
    Monitoring           :p2w3, 2026-03-23, 7d
    Advanced Features    :p2w4, 2026-03-30, 7d
    
    section Phase 3: Integration
    CLI Integration      :p3w1, 2026-04-06, 7d
    Hook Migration       :p3w2, 2026-04-13, 7d
    Integration Testing  :p3w3, 2026-04-20, 7d
    Launch               :p3w4, 2026-04-27, 7d
```

## State Machine: Template Lifecycle

```mermaid
stateDiagram-v2
    [*] --> Created: New template
    Created --> InUse: Applied to project
    InUse --> Monitoring: Track usage
    Monitoring --> Analyzing: Weekly analysis
    
    Analyzing --> Monitoring: Insufficient data
    Analyzing --> ProposalGenerated: Thresholds met
    
    ProposalGenerated --> UnderReview: User reviews
    UnderReview --> Approved: User approves
    UnderReview --> Rejected: User rejects
    
    Rejected --> Monitoring: Continue monitoring
    Approved --> Evolving: Apply changes
    Evolving --> NewVersion: Version incremented
    NewVersion --> InUse: New version active
    
    InUse --> Deprecated: Low confidence
    Deprecated --> [*]: Archived
```

## Component Interaction

```mermaid
graph TB
    subgraph "Hooks Layer"
        H1[log-feedback.sh]
        H2[session-start.sh]
        H3[backup-session-log.sh]
    end
    
    subgraph "Scripts Layer"
        S1[confidence-calculator.sh]
        S2[template-evolver.py]
        S3[auto-context-recall.sh]
    end
    
    subgraph "Storage Layer"
        DB[(SQLite)]
        MD[Markdown Files]
    end
    
    subgraph "Config Layer"
        C1[feedback-patterns.json]
        C2[settings.json]
    end
    
    H1 -->|Reads| C1
    H1 -->|Writes| DB
    H1 -->|Writes| MD
    
    H2 -->|Reads| DB
    H2 -->|Reads| C2
    
    H3 -->|Reads| DB
    H3 -->|Writes| MD
    
    S1 -->|Reads| DB
    S1 -->|Writes| DB
    
    S2 -->|Reads| DB
    S2 -->|Generates| Proposals[Evolution Proposals]
    
    S3 -->|Searches| DB
    S3 -->|Returns| Context[Context Results]
    
    style DB fill:#ffe1e1
    style MD fill:#ffe1e1
    style C1 fill:#e1f5ff
    style C2 fill:#e1f5ff
```

## Confidence Score Distribution

```mermaid
graph LR
    subgraph "Score Ranges"
        A[0.0 - 0.3<br/>Low Confidence<br/>❌ Avoid]
        B[0.3 - 0.5<br/>Moderate<br/>⚠️ Caution]
        C[0.5 - 0.7<br/>Good<br/>✓ Safe]
        D[0.7 - 0.9<br/>High<br/>✓✓ Recommended]
        E[0.9 - 1.0<br/>Very High<br/>⭐ Strongly Recommended]
    end
    
    style A fill:#ffcdd2
    style B fill:#fff9c4
    style C fill:#c8e6c9
    style D fill:#a5d6a7
    style E fill:#81c784
```

## Usage Instructions

### Viewing Diagrams

These Mermaid diagrams will render automatically in:
- GitHub (native support)
- GitLab (native support)
- VS Code (with Mermaid extension)
- Many Markdown viewers

### Editing Diagrams

1. **Online Editor**: https://mermaid.live/
2. **VS Code**: Install "Markdown Preview Mermaid Support" extension
3. **CLI**: Install `@mermaid-js/mermaid-cli` for PNG/SVG export

### Exporting to Images

```bash
# Install mermaid-cli
npm install -g @mermaid-js/mermaid-cli

# Export to PNG
mmdc -i docs/MERMAID_DIAGRAMS.md -o diagrams/

# Export to SVG
mmdc -i docs/MERMAID_DIAGRAMS.md -o diagrams/ -t svg
```

## References

- Mermaid Documentation: https://mermaid.js.org/
- Mermaid Live Editor: https://mermaid.live/
- GitHub Mermaid Support: https://github.blog/2022-02-14-include-diagrams-markdown-files-mermaid/

---

**Note**: These diagrams complement the text-based diagrams in [DIAGRAMS.md](DIAGRAMS.md). Use whichever format works best for your needs!
