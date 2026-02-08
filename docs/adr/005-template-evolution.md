# ADR-005: Template Evolution System

## Status

Accepted (2024)

## Context

Project templates (scaffolding, boilerplates) need to improve over time based on:
- What developers actually add to projects
- Which patterns lead to successful outcomes
- Common modifications across multiple projects
- User feedback on template-generated code

Static templates become outdated quickly. We need templates that evolve based on real-world usage.

## Decision

Implement a **data-driven template evolution system** that:

### Core Mechanism

1. **Track Template Usage**
   - Log when templates are applied to projects
   - Record template ID and version
   - Associate with project metadata

2. **Monitor Modifications**
   - Detect when users add directories/files to template-based projects
   - Track adoption rate (% of projects that add same thing)
   - Identify common patterns across projects

3. **Collect Outcome Data**
   - Link user feedback (wins/losses) to templates
   - Calculate win rate per template
   - Track confidence scores over time

4. **Generate Evolution Proposals**
   - Analyze patterns that meet thresholds
   - Propose additions to template
   - Require approval before applying

### Evolution Thresholds

```python
ADOPTION_THRESHOLD = 0.70   # 70% of projects must adopt
WIN_RATE_THRESHOLD = 0.75   # 75% win rate required
CONFIDENCE_THRESHOLD = 0.70 # 70% confidence score
MIN_SAMPLE_SIZE = 10        # Minimum projects analyzed
```

### Evolution Workflow

```
Template Applied to Project
    ↓
User Modifies Project (adds files/dirs)
    ↓
Modifications Logged
    ↓
User Provides Feedback (win/loss)
    ↓
Outcome Associated with Template
    ↓
Periodic Analysis (weekly)
    ↓
Calculate Adoption Rates & Win Rates
    ↓
Generate Evolution Proposals (if thresholds met)
    ↓
User Reviews Proposals (/review-templates)
    ↓
Approved Changes Applied to Template
    ↓
New Template Version Created
```

### Data Collection

**Template Usage Log** (`template-usage.jsonl`):
```json
{
  "timestamp": "2024-01-15T10:30:00Z",
  "template_id": "react-typescript-app",
  "template_version": "1.0.0",
  "project_path": "/Users/dev/my-app"
}
```

**Template Modifications Log** (`template-modifications.jsonl`):
```json
{
  "timestamp": "2024-01-15T11:00:00Z",
  "template_id": "react-typescript-app",
  "project_path": "/Users/dev/my-app",
  "type": "directory_added",
  "path": "src/hooks"
}
```

**Template Outcomes Log** (`template-outcomes.jsonl`):
```json
{
  "timestamp": "2024-01-15T12:00:00Z",
  "template_id": "react-typescript-app",
  "project_path": "/Users/dev/my-app",
  "outcome": "win"
}
```

### Evolution Proposal Format

```json
{
  "proposal_id": "PROP-0001",
  "template_id": "react-typescript-app",
  "current_version": "1.0.0",
  "proposed_version": "1.1.0",
  "changes": [
    {
      "type": "directory",
      "path": "src/hooks",
      "adoption_rate": 0.85,
      "projects_count": 17
    },
    {
      "type": "file",
      "path": "src/utils/api.ts",
      "adoption_rate": 0.72,
      "projects_count": 14
    }
  ],
  "rationale": "Based on analysis of 20 projects with 82% win rate and 78% confidence. 2 patterns adopted by ≥70% of projects.",
  "metrics": {
    "total_projects_analyzed": 20,
    "win_rate": 0.82,
    "confidence_score": 0.78,
    "changes_count": 2
  },
  "status": "pending",
  "created_at": "2024-01-20T10:00:00Z"
}
```

## Consequences

### Positive

- **Data-Driven**: Evolution based on actual usage, not assumptions
- **Quality Control**: High thresholds ensure only proven patterns promoted
- **Transparency**: Users see why changes are proposed
- **Approval Required**: No automatic changes without review
- **Versioned**: Template versions track evolution history
- **Measurable**: Clear metrics for template effectiveness
- **Continuous Improvement**: Templates get better over time

### Negative

- **Cold Start**: New templates have no data initially
- **Sample Size**: Need enough projects for statistical significance
- **Bias**: Popular patterns may not be best patterns
- **Maintenance**: Proposals need manual review
- **Complexity**: More moving parts than static templates
- **Storage**: Logs accumulate over time

### Mitigations

- **Default Templates**: Start with well-designed defaults
- **Minimum Thresholds**: Require minimum sample sizes
- **Multiple Metrics**: Use adoption + win rate + confidence
- **Review Process**: Human approval required
- **Documentation**: Clear rationale for each proposal
- **Cleanup**: Automated log rotation

## Alternatives Considered

### 1. Static Templates Only
- **Pros**: Simple, predictable, no complexity
- **Cons**: Become outdated, don't learn from usage
- **Rejected**: Doesn't meet goal of continuous improvement

### 2. Manual Template Updates
- **Pros**: Full control, human judgment
- **Cons**: Slow, subjective, doesn't scale
- **Rejected**: Too slow, misses patterns

### 3. Automatic Evolution (no approval)
- **Pros**: Fully automated, fast evolution
- **Cons**: Risky, could promote bad patterns, no oversight
- **Rejected**: Too risky without human review

### 4. Community Voting
- **Pros**: Democratic, crowd wisdom
- **Cons**: Popularity ≠ quality, coordination overhead
- **Rejected**: Too complex for initial version

### 5. A/B Testing Templates
- **Pros**: Scientific, controlled experiments
- **Cons**: Complex, requires large sample sizes, slow
- **Rejected**: Overkill for initial version

## Implementation Notes

### Tracking Template Usage

```bash
# When template applied
template_id="react-typescript-app"
template_version="1.0.0"
project_path="$PWD"

echo "{\"timestamp\":\"$(date -u +%Y-%m-%dT%H:%M:%SZ)\",\"template_id\":\"$template_id\",\"template_version\":\"$template_version\",\"project_path\":\"$project_path\"}" >> template-usage.jsonl

# Create metadata file in project
cat > .claude/template-metadata.json <<EOF
{
  "template_id": "$template_id",
  "template_version": "$template_version",
  "applied_at": "$(date -u +%Y-%m-%dT%H:%M:%SZ)"
}
EOF
```

### Detecting Modifications

```bash
# Compare current structure to template baseline
diff -r template-baseline/ project/ | grep "Only in project" | \
  while read line; do
    # Log addition
    echo "{\"timestamp\":\"$(date -u +%Y-%m-%dT%H:%M:%SZ)\",\"template_id\":\"$template_id\",\"type\":\"added\",\"path\":\"$path\"}" >> template-modifications.jsonl
  done
```

### Running Evolution Analysis

```bash
# Weekly cron job
0 2 * * 0 ~/.claude/scripts/template-evolver.py
```

### Reviewing Proposals

```bash
# User command
/review-templates

# Shows proposals with metrics
# User can approve/reject each
```

## Evolution Metrics

### Template Health Indicators

- **Usage Count**: How many times template applied
- **Win Rate**: % of projects with positive feedback
- **Confidence Score**: Overall confidence in template
- **Adoption Rate**: % of projects that add same patterns
- **Project Age**: How long projects survive (proxy for success)

### Proposal Quality Metrics

- **Adoption Rate**: Must be ≥70%
- **Win Rate**: Must be ≥75%
- **Confidence**: Must be ≥70%
- **Sample Size**: Must have ≥10 projects

## Security Considerations

- **Code Review**: All proposals reviewed before applying
- **Malicious Patterns**: Could promote security vulnerabilities
- **Privacy**: Project paths logged (consider anonymization)
- **Approval Required**: No automatic code changes

## Future Enhancements

- **ML-Based Analysis**: Use ML to identify subtle patterns
- **Semantic Similarity**: Group similar modifications
- **Deprecation Detection**: Identify patterns being removed
- **Cross-Template Learning**: Learn patterns across templates
- **Community Templates**: Share evolution data (opt-in)
- **Automated Testing**: Test evolved templates automatically
- **Rollback**: Easy rollback if evolution causes issues

## References

- `scripts/template-evolver.py`: Evolution analysis implementation
- `scripts/confidence-calculator.sh`: Confidence scoring
- `commands/review-templates.md`: User review interface
- Template Evolution Research: https://arxiv.org/abs/2103.07579

## Review Date

Review after 6 months to assess:
- Quality of evolution proposals
- Accuracy of thresholds
- User satisfaction with evolved templates
- Need for more sophisticated analysis
