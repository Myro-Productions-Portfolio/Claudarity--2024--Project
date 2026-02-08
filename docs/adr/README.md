# Architecture Decision Records (ADRs)

This directory contains Architecture Decision Records for Claudarity V2.0, documenting the key technical decisions made during the development of this reinforcement learning-based memory system for Claude Code.

## What is an ADR?

An Architecture Decision Record (ADR) captures an important architectural decision made along with its context and consequences. Each ADR describes:

- **Status**: Proposed, Accepted, Deprecated, or Superseded
- **Context**: The issue motivating this decision
- **Decision**: The change being proposed or implemented
- **Consequences**: The resulting context after applying the decision

## Index of ADRs

1. [ADR-001: Reinforcement Learning Approach](001-reinforcement-learning-approach.md)
2. [ADR-002: SQLite as Primary Storage](002-sqlite-storage.md)
3. [ADR-003: Bash Hooks for Event System](003-bash-hooks-event-system.md)
4. [ADR-004: Pattern-Based Feedback Detection](004-pattern-based-feedback.md)
5. [ADR-005: Template Evolution System](005-template-evolution.md)
6. [ADR-006: Dual Storage Strategy](006-dual-storage-strategy.md)
7. [ADR-007: Confidence Score Calculation](007-confidence-scoring.md)
8. [ADR-008: Background Processing Model](008-background-processing.md)

## ADR Lifecycle

- **Proposed**: Under discussion
- **Accepted**: Approved and implemented
- **Deprecated**: No longer recommended but still in use
- **Superseded**: Replaced by a newer decision (reference included)

## Creating New ADRs

When documenting a new architectural decision:

1. Copy the template from `adr-template.md`
2. Number it sequentially (ADR-XXX)
3. Fill in all sections
4. Update this index
5. Submit for review
