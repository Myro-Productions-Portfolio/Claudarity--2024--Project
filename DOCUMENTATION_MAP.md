# Claudarity Documentation Map

Quick reference guide to all documentation.

## 🚀 Start Here

**New to Claudarity?** Follow this path:

```
1. README.md
   ↓
2. docs/SUMMARY.md (Project Overview)
   ↓
3. docs/REINFORCEMENT_LEARNING.md (How it works)
   ↓
4. docs/FEATURES.md (What it can do)
```

## 📚 Documentation Structure

```
claudarity/
│
├── README.md ⭐                          # Start here
├── CLEANUP_COMPLETE.md                  # What we accomplished
├── DOCUMENTATION_MAP.md                 # This file
│
├── docs/
│   ├── INDEX.md                         # Complete documentation index
│   ├── SUMMARY.md ⭐                    # Project overview
│   ├── REINFORCEMENT_LEARNING.md ⭐     # How learning works
│   ├── CLEANUP_GUIDE.md                 # Improvement roadmap
│   ├── MIGRATION_PLAN.md                # 3-month plan
│   ├── DIAGRAMS.md                      # Text-based diagrams
│   ├── MERMAID_DIAGRAMS.md ⭐           # Interactive diagrams (GitHub/GitLab)
│   │
│   ├── ARCHITECTURE.md                  # System design
│   ├── FEATURES.md                      # Feature descriptions
│   ├── INSTALLATION.md                  # Setup guide
│   ├── TROUBLESHOOTING.md               # Common issues
│   │
│   └── adr/                             # Architecture Decision Records
│       ├── README.md                    # ADR index
│       ├── 001-reinforcement-learning-approach.md
│       ├── 002-sqlite-storage.md
│       ├── 003-bash-hooks-event-system.md
│       ├── 004-pattern-based-feedback.md
│       ├── 005-template-evolution.md
│       ├── 006-dual-storage-strategy.md
│       ├── 007-confidence-scoring.md
│       └── 008-background-processing.md
│
└── CONTRIBUTING.md                      # How to contribute
```

## 🎯 Documentation by Purpose

### I want to understand what this is

- **[README.md](README.md)** - Quick overview
- **[docs/SUMMARY.md](docs/SUMMARY.md)** - Complete project summary
- **[docs/REINFORCEMENT_LEARNING.md](docs/REINFORCEMENT_LEARNING.md)** - Learning approach

### I want to use it

- **[docs/FEATURES.md](docs/FEATURES.md)** - What it can do
- **[docs/INSTALLATION.md](docs/INSTALLATION.md)** - How to install
- **[docs/TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md)** - Fix issues

### I want to understand the design

- **[docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)** - System design
- **[docs/DIAGRAMS.md](docs/DIAGRAMS.md)** - Visual diagrams
- **[docs/adr/](docs/adr/)** - Design decisions

### I want to contribute

- **[CONTRIBUTING.md](CONTRIBUTING.md)** - Contribution guidelines
- **[docs/CLEANUP_GUIDE.md](docs/CLEANUP_GUIDE.md)** - What needs work
- **[docs/MIGRATION_PLAN.md](docs/MIGRATION_PLAN.md)** - Roadmap

### I want to see the big picture

- **[docs/INDEX.md](docs/INDEX.md)** - Complete documentation index
- **[CLEANUP_COMPLETE.md](CLEANUP_COMPLETE.md)** - What we accomplished
- **[docs/SUMMARY.md](docs/SUMMARY.md)** - Project overview

## 🔍 Documentation by Topic

### Reinforcement Learning

- **[docs/REINFORCEMENT_LEARNING.md](docs/REINFORCEMENT_LEARNING.md)** - Complete explanation
- **[docs/adr/001-reinforcement-learning-approach.md](docs/adr/001-reinforcement-learning-approach.md)** - Design decision
- **[docs/adr/007-confidence-scoring.md](docs/adr/007-confidence-scoring.md)** - Scoring formula

### Storage & Data

- **[docs/adr/002-sqlite-storage.md](docs/adr/002-sqlite-storage.md)** - Database design
- **[docs/adr/006-dual-storage-strategy.md](docs/adr/006-dual-storage-strategy.md)** - SQLite + Markdown

### Event System

- **[docs/adr/003-bash-hooks-event-system.md](docs/adr/003-bash-hooks-event-system.md)** - Hook architecture
- **[docs/adr/008-background-processing.md](docs/adr/008-background-processing.md)** - Non-blocking design

### Feedback & Learning

- **[docs/adr/004-pattern-based-feedback.md](docs/adr/004-pattern-based-feedback.md)** - Detection algorithm
- **[docs/adr/007-confidence-scoring.md](docs/adr/007-confidence-scoring.md)** - Confidence calculation

### Template Evolution

- **[docs/adr/005-template-evolution.md](docs/adr/005-template-evolution.md)** - Evolution system
- **[docs/CLEANUP_GUIDE.md](docs/CLEANUP_GUIDE.md)** - Completion tasks

## 📊 Quick Stats

- **Total Documents**: 16
- **ADRs**: 8
- **Guides**: 6
- **Words**: ~30,000
- **Code Examples**: 50+
- **Diagrams**: 10+

## 🎓 Learning Paths

### Path 1: User (30 minutes)

1. [README.md](README.md) - 5 min
2. [docs/SUMMARY.md](docs/SUMMARY.md) - 10 min
3. [docs/FEATURES.md](docs/FEATURES.md) - 10 min
4. [docs/INSTALLATION.md](docs/INSTALLATION.md) - 5 min

### Path 2: Developer (2 hours)

1. [README.md](README.md) - 5 min
2. [docs/SUMMARY.md](docs/SUMMARY.md) - 15 min
3. [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) - 20 min
4. [docs/REINFORCEMENT_LEARNING.md](docs/REINFORCEMENT_LEARNING.md) - 30 min
5. [docs/adr/](docs/adr/) - 40 min (skim all ADRs)
6. [docs/CLEANUP_GUIDE.md](docs/CLEANUP_GUIDE.md) - 10 min

### Path 3: Contributor (3 hours)

1. Complete Path 2 (Developer) - 2 hours
2. [CONTRIBUTING.md](CONTRIBUTING.md) - 15 min
3. [docs/MIGRATION_PLAN.md](docs/MIGRATION_PLAN.md) - 20 min
4. [docs/CLEANUP_GUIDE.md](docs/CLEANUP_GUIDE.md) - 15 min
5. Pick a task and start coding! - 10 min

### Path 4: Architect (4 hours)

1. Complete Path 2 (Developer) - 2 hours
2. Read all ADRs in detail - 1.5 hours
3. [docs/DIAGRAMS.md](docs/DIAGRAMS.md) - 20 min
4. [docs/MIGRATION_PLAN.md](docs/MIGRATION_PLAN.md) - 10 min

## 🔗 External Resources

### Reinforcement Learning

- Sutton & Barto: "Reinforcement Learning: An Introduction"
- Multi-Armed Bandits: https://en.wikipedia.org/wiki/Multi-armed_bandit
- Q-Learning: https://en.wikipedia.org/wiki/Q-learning

### Technologies Used

- SQLite: https://www.sqlite.org/docs.html
- FTS5: https://www.sqlite.org/fts5.html
- Bash: https://www.gnu.org/software/bash/manual/

### Best Practices

- Google Shell Style Guide: https://google.github.io/styleguide/shellguide.html
- Architecture Decision Records: https://adr.github.io/

## 📝 Document Status

| Status | Meaning |
|--------|---------|
| ⭐ | Start here / Most important |
| ✅ | Complete and current |
| ⚠️ | Needs update |
| 🚧 | Work in progress |

## 🤝 Contributing to Documentation

Found an error? Want to improve documentation?

1. Read [CONTRIBUTING.md](CONTRIBUTING.md)
2. Pick a document to improve
3. Submit a pull request
4. Help others learn!

## 📞 Getting Help

- **Questions**: Open an issue
- **Bugs**: Check [docs/TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md)
- **Ideas**: Start a discussion
- **Contributions**: See [CONTRIBUTING.md](CONTRIBUTING.md)

---

**Last Updated**: February 8, 2026
**Status**: Documentation complete ✅
**Next**: Begin stabilization phase (see [Migration Plan](docs/MIGRATION_PLAN.md))
