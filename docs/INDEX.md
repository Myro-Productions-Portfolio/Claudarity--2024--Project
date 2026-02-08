# Claudarity Documentation Index

Complete guide to all Claudarity documentation.

## Quick Start

New to Claudarity? Start here:

1. **[Project Summary](SUMMARY.md)** - What is this project?
2. **[README](../README.md)** - Quick overview and installation
3. **[Installation Guide](INSTALLATION.md)** - Detailed setup instructions
4. **[Features Guide](FEATURES.md)** - What can it do?

## Understanding the System

### Core Concepts

- **[System Diagrams](DIAGRAMS.md)** - Text-based visual representations
- **[Mermaid Diagrams](MERMAID_DIAGRAMS.md)** - Interactive diagrams (render in GitHub/GitLab)
- **[Reinforcement Learning](REINFORCEMENT_LEARNING.md)** - How the learning system works
  - Reward signals (wins/losses)
  - Confidence scoring
  - Template evolution
  - Learning algorithms

- **[Architecture Overview](ARCHITECTURE.md)** - System design
  - Storage layer (SQLite + Markdown)
  - Event system (Bash hooks)
  - Processing layer (Scripts)
  - Interface layer (Commands)

### Architecture Decision Records (ADRs)

Detailed technical decisions and rationale:

1. **[ADR-001: Reinforcement Learning Approach](adr/001-reinforcement-learning-approach.md)**
   - Why reinforcement learning?
   - Win/loss feedback mechanism
   - Confidence scoring
   - Template evolution

2. **[ADR-002: SQLite as Primary Storage](adr/002-sqlite-storage.md)**
   - Why SQLite?
   - Database schema
   - Performance characteristics
   - Backup strategy

3. **[ADR-003: Bash Hooks for Event System](adr/003-bash-hooks-event-system.md)**
   - Why bash hooks?
   - Event-driven architecture
   - Hook types and patterns
   - Non-blocking design

4. **[ADR-004: Pattern-Based Feedback Detection](adr/004-pattern-based-feedback.md)**
   - Why pattern matching?
   - Detection algorithm
   - Accuracy metrics
   - Future ML integration

5. **[ADR-005: Template Evolution System](adr/005-template-evolution.md)**
   - Why evolve templates?
   - Evolution workflow
   - Thresholds and criteria
   - Approval process

6. **[ADR-006: Dual Storage Strategy](adr/006-dual-storage-strategy.md)**
   - Why both SQLite and Markdown?
   - Synchronization approach
   - Backup and recovery
   - Trade-offs

7. **[ADR-007: Confidence Score Calculation](adr/007-confidence-scoring.md)**
   - Multi-factor scoring formula
   - Weight rationale
   - Tuning guidelines
   - Validation metrics

8. **[ADR-008: Background Processing Model](adr/008-background-processing.md)**
   - Fork-to-background pattern
   - Non-blocking design
   - Error handling
   - Resource management

## Using Claudarity

### User Guides

- **[Features Guide](FEATURES.md)** - Complete feature descriptions
  - Feedback learning
  - Context memory
  - Template evolution
  - Session management

- **[Installation Guide](INSTALLATION.md)** - Setup instructions
  - Requirements
  - Installation steps
  - Configuration
  - Verification

- **[Troubleshooting](TROUBLESHOOTING.md)** - Common issues
  - Installation problems
  - Runtime errors
  - Performance issues
  - Data recovery

### Developer Guides

- **[Contributing Guide](../CONTRIBUTING.md)** - How to contribute
  - Development setup
  - Code guidelines
  - Testing
  - Pull request process

- **[Cleanup Guide](CLEANUP_GUIDE.md)** - Improvement tasks
  - What needs fixing
  - Code cleanup tasks
  - Testing infrastructure
  - Modernization steps

- **[Migration Plan](MIGRATION_PLAN.md)** - Roadmap to production
  - Phase 1: Stabilization
  - Phase 2: Modernization
  - Phase 3: Integration
  - Timeline and milestones

## Technical Deep Dives

### Learning System

- **[Reinforcement Learning](REINFORCEMENT_LEARNING.md)**
  - Core concepts
  - Learning mechanisms
  - Mathematical formulation
  - Comparison to traditional RL

### Storage System

- **[ADR-002: SQLite Storage](adr/002-sqlite-storage.md)**
  - Database schema
  - Query patterns
  - Performance optimization
  - Backup strategy

- **[ADR-006: Dual Storage](adr/006-dual-storage-strategy.md)**
  - SQLite + Markdown approach
  - Synchronization
  - Recovery procedures

### Event System

- **[ADR-003: Bash Hooks](adr/003-bash-hooks-event-system.md)**
  - Hook types
  - Event flow
  - Hook development

- **[ADR-008: Background Processing](adr/008-background-processing.md)**
  - Fork-to-background pattern
  - Error handling
  - Resource management

### Feedback System

- **[ADR-004: Pattern Detection](adr/004-pattern-based-feedback.md)**
  - Detection algorithm
  - Pattern configuration
  - Accuracy metrics

- **[ADR-007: Confidence Scoring](adr/007-confidence-scoring.md)**
  - Scoring formula
  - Factor definitions
  - Tuning guidelines

### Template System

- **[ADR-005: Template Evolution](adr/005-template-evolution.md)**
  - Evolution workflow
  - Adoption tracking
  - Proposal generation
  - Approval process

## Project Status & Planning

### Current State

- **[Project Summary](SUMMARY.md)** - Complete overview
  - What was built
  - What works well
  - What needs improvement
  - Lessons learned

- **[README](../README.md)** - Current status
  - Working features
  - Known issues
  - Requirements

### Future Plans

- **[Cleanup Guide](CLEANUP_GUIDE.md)** - Immediate improvements
  - Remove hardcoded paths
  - Add testing
  - Improve error handling
  - Add ML integration

- **[Migration Plan](MIGRATION_PLAN.md)** - Long-term roadmap
  - 3-month plan
  - Phase-by-phase approach
  - Success criteria
  - Risk management

## Reference Materials

### Code Organization

```
claudarity/
├── hooks/              # Event hooks (bash)
│   ├── log-feedback.sh
│   ├── session-start.sh
│   └── ...
├── scripts/            # Utility scripts
│   ├── confidence-calculator.sh
│   ├── template-evolver.py
│   └── ...
├── commands/           # Slash commands
│   ├── baseline.md
│   ├── gomemory.md
│   └── ...
├── config/             # Configuration
│   ├── feedback-patterns.json
│   └── settings.json
└── docs/               # Documentation
    ├── adr/            # Architecture decisions
    ├── SUMMARY.md
    └── ...
```

### Key Files

**Hooks**:
- `hooks/log-feedback.sh` - Feedback detection
- `hooks/session-start.sh` - Session initialization
- `hooks/backup-session-log.sh` - Session backup

**Scripts**:
- `scripts/confidence-calculator.sh` - Confidence scoring
- `scripts/template-evolver.py` - Template evolution
- `scripts/auto-context-recall.sh` - Context retrieval
- `scripts/init-claudarity-db.sh` - Database initialization

**Configuration**:
- `config/feedback-patterns.json` - Feedback patterns
- `config/settings.json` - System settings

**Documentation**:
- `docs/SUMMARY.md` - Project overview
- `docs/ARCHITECTURE.md` - System design
- `docs/REINFORCEMENT_LEARNING.md` - Learning approach
- `docs/adr/` - Architecture decisions

## Getting Help

### Documentation

1. Check this index for relevant docs
2. Read the [Troubleshooting Guide](TROUBLESHOOTING.md)
3. Review [ADRs](adr/) for design decisions

### Community

- **Issues**: Report bugs and request features
- **Discussions**: Ask questions and share ideas
- **Contributing**: See [Contributing Guide](../CONTRIBUTING.md)

### Support

- Check logs: `~/.claude/logs/`
- Run diagnostics: `~/.claude/scripts/test-all.sh`
- Review database: `sqlite3 ~/.claude/claudarity.db`

## Document Status

| Document | Status | Last Updated |
|----------|--------|--------------|
| README.md | ✅ Current | 2026-02 |
| SUMMARY.md | ✅ Current | 2026-02 |
| ARCHITECTURE.md | ✅ Current | 2024 |
| FEATURES.md | ✅ Current | 2024 |
| REINFORCEMENT_LEARNING.md | ✅ Current | 2026-02 |
| CLEANUP_GUIDE.md | ✅ Current | 2026-02 |
| MIGRATION_PLAN.md | ✅ Current | 2026-02 |
| ADR-001 | ✅ Current | 2026-02 |
| ADR-002 | ✅ Current | 2026-02 |
| ADR-003 | ✅ Current | 2026-02 |
| ADR-004 | ✅ Current | 2026-02 |
| ADR-005 | ✅ Current | 2026-02 |
| ADR-006 | ✅ Current | 2026-02 |
| ADR-007 | ✅ Current | 2026-02 |
| ADR-008 | ✅ Current | 2026-02 |
| INSTALLATION.md | ⚠️ Needs update | 2024 |
| TROUBLESHOOTING.md | ⚠️ Needs update | 2024 |
| CONTRIBUTING.md | ✅ Current | 2024 |

## Contributing to Documentation

Documentation improvements welcome! See [Contributing Guide](../CONTRIBUTING.md).

**Priority areas**:
- Update INSTALLATION.md with current setup
- Expand TROUBLESHOOTING.md with common issues
- Add code examples to ADRs
- Create video tutorials
- Translate to other languages

---

**Last Updated**: February 2026
**Maintainer**: Myro Productions
**Status**: Documentation complete, ready for modernization phase
