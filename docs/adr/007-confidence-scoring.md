# ADR-007: Confidence Score Calculation

## Status

Accepted (2024)

## Context

The reinforcement learning system needs a way to quantify how confident it is in templates, patterns, and recommendations. This confidence should:
- Reflect actual success rate (win/loss ratio)
- Account for usage frequency (more data = more confidence)
- Consider recency (recent performance matters more)
- Factor in maturity (established patterns more reliable)

A single metric is needed that combines these factors into an actionable score.

## Decision

Implement a **weighted multi-factor confidence score** calculated as:

```python
confidence_score = (
    usage_factor * 0.3 +      # 30% weight
    win_rate * 0.4 +          # 40% weight (most important)
    recency_factor * 0.1 +    # 10% weight
    maturity_factor * 0.2     # 20% weight
)
```

### Factor Definitions

#### 1. Usage Factor (30%)
```python
usage_factor = min(application_count / 10, 1.0)
```

**Purpose**: Confidence increases with more data points

**Rationale**: 
- Need minimum sample size for statistical significance
- 10 applications = full confidence in this factor
- Prevents premature high confidence from lucky early wins

**Example**:
- 1 application: 0.1 usage factor
- 5 applications: 0.5 usage factor
- 10+ applications: 1.0 usage factor

#### 2. Win Rate (40%)
```python
win_rate = wins / (wins + losses) if (wins + losses) > 0 else 0.5
```

**Purpose**: Primary indicator of success

**Rationale**:
- Most important factor (40% weight)
- Direct measure of user satisfaction
- Default to neutral 0.5 if no feedback

**Example**:
- 8 wins, 2 losses: 0.8 win rate
- 5 wins, 5 losses: 0.5 win rate
- 2 wins, 8 losses: 0.2 win rate

#### 3. Recency Factor (10%)
```python
recency_factor = recent_wins / recent_total if recent_total > 0 else 0.0

# Where "recent" = last 90 days
```

**Purpose**: Recent performance matters more than old data

**Rationale**:
- User preferences change over time
- Recent feedback more relevant
- Helps system adapt to changes

**Example**:
- Last 90 days: 4 wins, 1 loss → 0.8 recency
- Last 90 days: 1 win, 4 losses → 0.2 recency
- No recent data → 0.0 recency

#### 4. Maturity Factor (20%)
```python
maturity_factor = min(avg_project_age_days / 90, 1.0)
```

**Purpose**: Patterns that lead to long-lived projects are better

**Rationale**:
- Projects that survive indicate success
- 90 days = considered mature
- Proxy for long-term satisfaction

**Example**:
- Projects avg 30 days old: 0.33 maturity
- Projects avg 90+ days old: 1.0 maturity
- New projects: 0.0 maturity

### Score Interpretation

```
0.0 - 0.3: Low confidence (avoid using)
0.3 - 0.5: Moderate confidence (use with caution)
0.5 - 0.7: Good confidence (safe to use)
0.7 - 0.9: High confidence (recommended)
0.9 - 1.0: Very high confidence (strongly recommended)
```

## Consequences

### Positive

- **Balanced**: No single factor dominates
- **Interpretable**: Clear meaning of each component
- **Actionable**: Easy to understand what improves confidence
- **Adaptive**: Responds to changing patterns
- **Fair**: Requires multiple positive signals for high confidence
- **Transparent**: Users can see why confidence is high/low

### Negative

- **Arbitrary Weights**: Weights chosen empirically, not scientifically
- **Linear Combination**: May not capture complex interactions
- **Fixed Thresholds**: 10 applications, 90 days are arbitrary
- **Cold Start**: New items start with low confidence
- **Sensitivity**: Small changes in factors can affect score

### Mitigations

- **Tunable Weights**: Weights can be adjusted based on data
- **Minimum Thresholds**: Require minimum samples before using
- **Default Values**: Reasonable defaults for missing data
- **Monitoring**: Track confidence accuracy over time
- **User Override**: Users can adjust weights in config

## Alternatives Considered

### 1. Win Rate Only
```python
confidence = wins / (wins + losses)
```

- **Pros**: Simple, direct measure
- **Cons**: Ignores sample size, recency, maturity
- **Rejected**: Too simplistic, unreliable with small samples

### 2. Bayesian Confidence Interval
```python
# Wilson score interval
confidence = lower_bound_of_wilson_score(wins, total)
```

- **Pros**: Statistically rigorous, handles small samples well
- **Cons**: Complex, doesn't account for recency or maturity
- **Rejected**: Too complex for initial version

### 3. Thompson Sampling
```python
# Sample from Beta distribution
confidence = beta_distribution(wins + 1, losses + 1).mean()
```

- **Pros**: Probabilistic, good for exploration/exploitation
- **Cons**: Complex, requires sampling, doesn't use all factors
- **Rejected**: Overkill for this use case

### 4. Exponential Moving Average
```python
# Decay old data exponentially
confidence = EMA(outcomes, decay=0.9)
```

- **Pros**: Naturally handles recency
- **Cons**: Doesn't account for sample size or maturity
- **Rejected**: Doesn't capture all factors

### 5. Machine Learning Model
```python
# Train model to predict success
confidence = ml_model.predict(features)
```

- **Pros**: Can learn complex patterns
- **Cons**: Requires training data, black box, complex
- **Rejected**: Too complex for initial version

## Implementation Notes

### Calculation Script

```bash
# scripts/confidence-calculator.sh

# Read usage data
usage_count=$(sqlite3 "$DB" "SELECT COUNT(*) FROM template_usage WHERE template_id = '$id';")

# Read outcomes
wins=$(sqlite3 "$DB" "SELECT COUNT(*) FROM feedback WHERE template_id = '$id' AND type = 'win';")
losses=$(sqlite3 "$DB" "SELECT COUNT(*) FROM feedback WHERE template_id = '$id' AND type = 'loss';")

# Calculate factors
usage_factor=$(echo "scale=4; if ($usage_count >= 10) 1.0 else $usage_count / 10" | bc)
win_rate=$(echo "scale=4; if ($wins + $losses > 0) $wins / ($wins + $losses) else 0.5" | bc)

# Calculate confidence
confidence=$(echo "scale=4; $usage_factor * 0.3 + $win_rate * 0.4 + $recency_factor * 0.1 + $maturity_factor * 0.2" | bc)
```

### Storage

```sql
CREATE TABLE template_confidence (
    template_id TEXT PRIMARY KEY,
    confidence_score REAL NOT NULL,
    application_count INTEGER NOT NULL,
    win_rate REAL NOT NULL,
    recency_factor REAL NOT NULL,
    maturity_factor REAL NOT NULL,
    last_calculated TEXT NOT NULL
);
```

### Usage

```bash
# Get confidence for template
confidence=$(sqlite3 "$DB" "SELECT confidence_score FROM template_confidence WHERE template_id = '$id';")

# Use in decision making
if (( $(echo "$confidence >= 0.7" | bc -l) )); then
  echo "High confidence - recommend this template"
else
  echo "Low confidence - suggest alternatives"
fi
```

## Tuning Guidelines

### Adjusting Weights

**Increase win_rate weight** if:
- User feedback is very reliable
- Want to prioritize proven patterns
- Have enough data for accurate win rates

**Increase usage_factor weight** if:
- Want to favor well-tested patterns
- Have concerns about small sample sizes
- Want more conservative recommendations

**Increase recency_factor weight** if:
- User preferences change frequently
- Want to adapt quickly to new patterns
- Recent data is more reliable

**Increase maturity_factor weight** if:
- Long-term success is important
- Want to favor stable patterns
- Have good project age data

### Adjusting Thresholds

**Usage threshold** (default 10):
- Increase if need more data for confidence
- Decrease if data is scarce
- Consider domain (complex = higher threshold)

**Recency window** (default 90 days):
- Increase for stable domains
- Decrease for fast-changing domains
- Consider user activity level

**Maturity threshold** (default 90 days):
- Increase for long-term projects
- Decrease for short-term projects
- Consider project lifecycle

## Validation

### Metrics to Track

1. **Calibration**: Do 70% confidence items succeed 70% of time?
2. **Discrimination**: Do high confidence items succeed more than low?
3. **Coverage**: What % of items have confidence scores?
4. **Stability**: How much do scores change over time?

### Expected Performance

- **Calibration error**: <10%
- **AUC-ROC**: >0.8
- **Coverage**: >90% of templates
- **Stability**: <20% change per week

## Future Enhancements

### 1. Dynamic Weights
Learn optimal weights from data:
```python
# Optimize weights to maximize calibration
weights = optimize(lambda w: calibration_error(w), initial_weights)
```

### 2. Confidence Intervals
Provide uncertainty estimates:
```python
confidence_mean = 0.75
confidence_std = 0.05
confidence_interval = (0.70, 0.80)  # 95% CI
```

### 3. Multi-Dimensional Confidence
Different confidence for different aspects:
```python
confidence = {
    'correctness': 0.85,
    'style': 0.70,
    'performance': 0.60,
    'overall': 0.72
}
```

### 4. Contextual Confidence
Confidence varies by context:
```python
confidence = calculate_confidence(template, context={
    'project_type': 'web_app',
    'team_size': 'small',
    'experience': 'intermediate'
})
```

### 5. Confidence Decay
Reduce confidence over time without updates:
```python
# Decay 10% per month without new data
confidence_current = confidence_base * (0.9 ** months_since_update)
```

## References

- `scripts/confidence-calculator.sh`: Implementation
- Wilson Score Interval: https://en.wikipedia.org/wiki/Binomial_proportion_confidence_interval
- Thompson Sampling: https://en.wikipedia.org/wiki/Thompson_sampling
- Calibration: https://en.wikipedia.org/wiki/Calibration_(statistics)

## Review Date

Review quarterly to assess:
- Calibration accuracy
- Weight effectiveness
- Threshold appropriateness
- Need for more sophisticated approach
