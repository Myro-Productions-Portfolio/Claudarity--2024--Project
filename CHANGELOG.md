# Changelog

All notable changes to the Claudarity project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Planned
- Remove hardcoded paths (see [Cleanup Guide](docs/CLEANUP_GUIDE.md))
- Add automated testing infrastructure
- Complete template evolution workflow
- Add ML-based sentiment analysis

## [2.0.0] - 2026-02-08

### Added - Documentation Phase
- **Architecture Decision Records**: 8 comprehensive ADRs documenting all major design decisions
  - ADR-001: Reinforcement Learning Approach
  - ADR-002: SQLite as Primary Storage
  - ADR-003: Bash Hooks for Event System
  - ADR-004: Pattern-Based Feedback Detection
  - ADR-005: Template Evolution System
  - ADR-006: Dual Storage Strategy
  - ADR-007: Confidence Score Calculation
  - ADR-008: Background Processing Model

- **Comprehensive Guides**:
  - Project Summary (docs/SUMMARY.md)
  - Cleanup Guide (docs/CLEANUP_GUIDE.md)
  - Migration Plan (docs/MIGRATION_PLAN.md)
  - Reinforcement Learning Explanation (docs/REINFORCEMENT_LEARNING.md)
  - System Diagrams (docs/DIAGRAMS.md)
  - Mermaid Diagrams (docs/MERMAID_DIAGRAMS.md)
  - Documentation Index (docs/INDEX.md)

- **Navigation Aids**:
  - Documentation Map (DOCUMENTATION_MAP.md)
  - Cleanup Completion Report (CLEANUP_COMPLETE.md)
  - Repository Audit (REPOSITORY_AUDIT.md)

- **Visual Documentation**:
  - 10+ text-based diagrams
  - 11 interactive Mermaid diagrams
  - System architecture visualizations
  - Workflow diagrams

### Changed
- Updated README.md with project status and experimental notice
- Enhanced CONTRIBUTING.md with clear guidelines
- Reorganized documentation structure

### Documented
- Complete reinforcement learning approach with mathematical formulations
- All design decisions with rationale and alternatives considered
- System architecture and component interactions
- Data flow and processing patterns
- Confidence scoring formula and tuning guidelines
- Template evolution workflow
- Background processing model
- Storage strategy and backup procedures

### Fixed
- Documentation organization and navigation
- Internal linking between documents
- Project status clarity

## [1.0.0] - 2024 (Estimated)

### Added - Initial Implementation
- **Core Features**:
  - Feedback learning system with win/loss tracking
  - Pattern-based feedback detection
  - Confidence score calculation
  - Context memory with SQLite storage
  - Full-text search (FTS5)
  - Template evolution system (partial)
  - Event hook system (bash-based)
  - Dual storage (SQLite + Markdown)

- **Hooks**:
  - UserPromptSubmit hook for feedback detection
  - SessionStart hook for initialization
  - Stop hook for session backup
  - Context detection and injection hooks

- **Scripts**:
  - confidence-calculator.sh
  - template-evolver.py
  - auto-context-recall.sh
  - init-claudarity-db.sh
  - 20+ utility scripts

- **Commands**:
  - /baseline - Win/loss statistics
  - /gomemory - Context search
  - /prefs - Query preferences
  - /review-templates - Template review
  - 10+ slash commands

- **Storage**:
  - SQLite database with FTS5
  - Markdown cache files
  - Session logging
  - Terminal activity capture

### Known Issues
- Hardcoded paths (not portable across systems)
- Template evolution workflow incomplete
- No automated testing
- Limited error handling in background processes
- Pattern detection accuracy ~77% (needs ML augmentation)

---

## Version History Summary

- **v2.0.0** (2026-02-08): Documentation and cleanup phase - Repository ready for GitHub
- **v1.0.0** (2024): Initial experimental implementation - Core features working

## Migration Notes

### From v1.0.0 to v2.0.0
- No breaking changes to functionality
- Added comprehensive documentation
- Identified issues and created improvement roadmap
- No code changes required for existing installations

### Future Breaking Changes (v3.0.0)
- Configuration file will replace hardcoded paths
- Template evolution workflow will be completed
- ML-based sentiment analysis will be added

See [Migration Plan](docs/MIGRATION_PLAN.md) for detailed upgrade path.

---

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines on contributing to this project.

## Links

- [Documentation](docs/INDEX.md)
- [Architecture Decision Records](docs/adr/)
- [Migration Plan](docs/MIGRATION_PLAN.md)
- [Cleanup Guide](docs/CLEANUP_GUIDE.md)
