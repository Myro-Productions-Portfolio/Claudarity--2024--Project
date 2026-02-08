# Reinforcement Learning in Claudarity

This document explains the reinforcement learning approach used in Claudarity V2.0 and how it enables the system to learn and improve over time.

## Overview

Claudarity implements a **reward-based reinforcement learning system** where:
- **Actions**: Claude's responses and template applications
- **Rewards**: User feedback (praise = positive, criticism = negative)
- **Learning**: Confidence scores and template evolution based on rewards
- **Goal**: Maximize positive feedback over time

## Core Concepts

### 1. Reward Signal

The system uses **binary reward classification**:

```
Reward = {
  +1  if user gives praise (win)
  -1  if user gives criticism (loss)
   0  if no feedback (neutral)
}
```

**Detection Method**: Pattern matching on user messages
- Praise patterns: "great job", "awesome", "perfect"
- Criticism patterns: "wrong", "terrible", "bad"

**Filtering**: Only short messages (≤5 words, ≤50 chars) to avoid false positives

### 2. State Representation

The system tracks multiple state dimensions:

```python
State = {
  'template_id': str,           # Which template was used
  'project_context': str,       # Project type and domain
  'user_preferences': dict,     # Learned preferences
  'confidence_scores': dict,    # Current confidence levels
  'recent_history': list        # Recent interactions
}
```

### 3. Action Space

Actions the system can take:

```python
Actions = {
  'select_template': Choose which template to recommend
  'apply_pattern': Apply learned coding pattern
  'inject_context': Include relevant historical context
  'adjust_confidence': Update confidence scores
}
```

### 4. Learning Algorithm

Claudarity uses a **multi-armed bandit** approach with **confidence scoring**:

```python
def calculate_confidence(template_id):
    """
    Confidence score formula:
    
    confidence = (
        usage_factor * 0.3 +      # How often used
        win_rate * 0.4 +          # Success rate
        recency_factor * 0.1 +    # Recent performance
        maturity_factor * 0.2     # Project age/stability
    )
    """
    usage_factor = min(usage_count / 10, 1.0)
    win_rate = wins / (wins + losses) if (wins + losses) > 0 else 0.5
    recency_factor = recent_wins / recent_total if recent_total > 0 else 0.0
    maturity_factor = avg_project_age / 90  # 90 days = mature
    
    return (
        usage_factor * 0.3 +
        win_rate * 0.4 +
        recency_factor * 0.1 +
        maturity_factor * 0.2
    )
```

### 5. Exploration vs Exploitation

The system balances:
- **Exploitation**: Use high-confidence templates/patterns
- **Exploration**: Try new or low-confidence options

```python
def select_template(templates, epsilon=0.1):
    """
    Epsilon-greedy selection:
    - 90% of time: Choose highest confidence (exploit)
    - 10% of time: Choose randomly (explore)
    """
    if random.random() < epsilon:
        return random.choice(templates)  # Explore
    else:
        return max(templates, key=lambda t: t.confidence)  # Exploit
```

## Learning Mechanisms

### 1. Immediate Feedback Learning

**Trigger**: User provides feedback on recent interaction

**Process**:
```
User: "great job!"
    ↓
Detect praise pattern
    ↓
Identify recent actions (template used, patterns applied)
    ↓
Increase confidence scores for those actions
    ↓
Log outcome for future analysis
```

**Update Rule**:
```python
# Simple moving average
new_confidence = (
    old_confidence * (n - 1) + reward
) / n

# Where:
# - old_confidence: Previous confidence score
# - reward: +1 for win, -1 for loss
# - n: Total number of outcomes
```

### 2. Template Evolution Learning

**Trigger**: Periodic analysis (weekly)

**Process**:
```
Analyze all template-based projects
    ↓
Identify common modifications (adoption rate)
    ↓
Calculate win rate for each template
    ↓
Generate evolution proposals for high-performing patterns
    ↓
Apply approved changes to templates
    ↓
New template version with improved defaults
```

**Evolution Criteria**:
```python
def should_evolve(pattern):
    """
    Pattern must meet ALL criteria:
    - Adoption rate ≥ 70% (most projects add it)
    - Win rate ≥ 75% (leads to success)
    - Confidence ≥ 70% (system is confident)
    - Sample size ≥ 10 (statistically significant)
    """
    return (
        pattern.adoption_rate >= 0.70 and
        pattern.win_rate >= 0.75 and
        pattern.confidence >= 0.70 and
        pattern.sample_size >= 10
    )
```

### 3. Preference Learning

**Trigger**: Continuous, based on all interactions

**Process**:
```
User provides feedback on code
    ↓
Extract code patterns from context
    ↓
Associate patterns with feedback
    ↓
Build preference profile
    ↓
Apply preferences to future responses
```

**Example**:
```python
# User consistently praises error handling with try/catch
# System learns preference:
preferences['error_handling'] = {
    'pattern': 'try/catch blocks',
    'confidence': 0.85,
    'usage_count': 12,
    'win_rate': 0.92
}

# Future responses automatically include try/catch
```

## Reinforcement Learning Cycle

### Complete Learning Loop

```
┌─────────────────────────────────────────────────────────┐
│                    1. INITIAL STATE                      │
│  - User asks for help                                   │
│  - System has current confidence scores                 │
│  - Historical context available                         │
└────────────────┬────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────┐
│                    2. ACTION SELECTION                   │
│  - Choose template (if applicable)                      │
│  - Select patterns to apply                             │
│  - Decide what context to inject                        │
│  - Balance exploration vs exploitation                  │
└────────────────┬────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────┐
│                    3. EXECUTE ACTION                     │
│  - Claude generates response                            │
│  - Apply selected template/patterns                     │
│  - Include relevant context                             │
│  - Log action taken                                     │
└────────────────┬────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────┐
│                    4. OBSERVE REWARD                     │
│  - User provides feedback (or not)                      │
│  - Detect praise/criticism patterns                     │
│  - Classify as win/loss/neutral                         │
│  - Log outcome with context                             │
└────────────────┬────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────┐
│                    5. UPDATE BELIEFS                     │
│  - Update confidence scores                             │
│  - Adjust template preferences                          │
│  - Update user preference profile                       │
│  - Log for future analysis                              │
└────────────────┬────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────┐
│                    6. PERIODIC LEARNING                  │
│  - Analyze patterns (weekly)                            │
│  - Generate evolution proposals                         │
│  - Update templates (if approved)                       │
│  - Refine confidence formulas                           │
└────────────────┬────────────────────────────────────────┘
                 │
                 └──────────► Back to step 1 (next interaction)
```

## Mathematical Formulation

### Confidence Score Function

```
C(t) = α·U(t) + β·W(t) + γ·R(t) + δ·M(t)

Where:
- C(t): Confidence score at time t
- U(t): Usage factor (normalized usage count)
- W(t): Win rate (wins / total outcomes)
- R(t): Recency factor (recent performance)
- M(t): Maturity factor (project age)
- α, β, γ, δ: Weights (sum to 1.0)

Default weights:
- α = 0.3 (usage)
- β = 0.4 (win rate - most important)
- γ = 0.1 (recency)
- δ = 0.2 (maturity)
```

### Win Rate Calculation

```
W(t) = Σ(rewards) / N

Where:
- rewards ∈ {-1, 0, +1}
- N = total number of outcomes
- Normalized to [0, 1] range
```

### Adoption Rate (for template evolution)

```
A(p) = |{projects that added pattern p}| / |{total projects}|

Evolution threshold: A(p) ≥ 0.70
```

### Expected Value (for action selection)

```
EV(a) = C(a) + ε·E(a)

Where:
- EV(a): Expected value of action a
- C(a): Confidence score for action a
- E(a): Exploration bonus
- ε: Exploration rate (default 0.1)

E(a) = √(2·ln(N) / n(a))

Where:
- N: Total actions taken
- n(a): Times action a was taken
- This is the UCB1 (Upper Confidence Bound) formula
```

## Learning Characteristics

### Convergence

The system converges to optimal behavior when:
1. **Sufficient data**: Enough feedback samples collected
2. **Consistent feedback**: User feedback is consistent
3. **Stable environment**: User preferences don't change drastically

**Typical convergence time**: 50-100 interactions per template/pattern

### Adaptation

The system adapts to changes through:
1. **Recency weighting**: Recent feedback weighted more heavily
2. **Confidence decay**: Old confidence scores decay over time
3. **Continuous exploration**: Always trying new options (10% of time)

### Robustness

The system is robust to:
1. **Noisy feedback**: Requires multiple samples before high confidence
2. **Missing feedback**: Neutral outcomes don't hurt confidence much
3. **False positives**: Short message filtering reduces noise
4. **Outliers**: Moving average smooths out anomalies

## Comparison to Traditional RL

### Similarities to Q-Learning

```python
# Traditional Q-Learning update
Q(s, a) = Q(s, a) + α·(r + γ·max(Q(s', a')) - Q(s, a))

# Claudarity confidence update (simplified)
C(template) = C(template) + α·(reward - C(template))

# Both:
# - Learn from rewards
# - Update value estimates
# - Balance exploration/exploitation
```

### Differences from Traditional RL

| Aspect | Traditional RL | Claudarity |
|--------|---------------|------------|
| State space | Discrete/continuous | Implicit (context-based) |
| Action space | Well-defined | Dynamic (templates/patterns) |
| Reward signal | Immediate | Delayed (user feedback) |
| Learning rate | Fixed α | Adaptive (based on sample size) |
| Exploration | ε-greedy or UCB | ε-greedy (10%) |
| Model | Model-free | Hybrid (some model-based) |

## Advantages of This Approach

### 1. Simplicity
- No complex neural networks
- Easy to understand and debug
- Fast computation (no training time)

### 2. Transparency
- Clear confidence scores
- Explainable decisions
- Users can see what's being learned

### 3. Efficiency
- Learns from sparse feedback
- No large training datasets needed
- Continuous online learning

### 4. Robustness
- Handles noisy feedback
- Adapts to changing preferences
- Graceful degradation

### 5. Privacy
- All learning happens locally
- No data sent to cloud
- User has full control

## Limitations

### 1. Cold Start Problem
- New templates have no confidence data
- Solution: Start with neutral 0.5 confidence

### 2. Sparse Rewards
- Users don't always provide feedback
- Solution: Use implicit signals (usage, project age)

### 3. Delayed Rewards
- Feedback may come much later
- Solution: Associate with recent actions

### 4. Non-Stationary Environment
- User preferences change over time
- Solution: Recency weighting and confidence decay

### 5. Limited Context
- Can't capture all nuances
- Solution: Combine with ML sentiment analysis

## Future Enhancements

### 1. Deep Reinforcement Learning
- Use neural networks for state representation
- Learn more complex patterns
- Better generalization

### 2. Multi-Armed Contextual Bandits
- Consider context when selecting actions
- Personalized recommendations
- Better cold start handling

### 3. Transfer Learning
- Learn from other users (opt-in)
- Share successful patterns
- Faster convergence

### 4. Active Learning
- Ask for feedback when uncertain
- Targeted exploration
- More efficient learning

### 5. Hierarchical RL
- Learn at multiple levels (templates, patterns, preferences)
- Compositional learning
- Better scalability

## Conclusion

Claudarity's reinforcement learning approach is:
- **Simple**: Easy to understand and implement
- **Effective**: Learns from real user feedback
- **Transparent**: Clear confidence scores and decisions
- **Privacy-preserving**: All learning happens locally
- **Adaptive**: Continuously improves over time

The system successfully demonstrates that reinforcement learning can be applied to code assistants without complex ML infrastructure, providing genuine improvement based on user behavior.

## References

### Academic Papers
- Sutton & Barto (2018): "Reinforcement Learning: An Introduction"
- Auer et al. (2002): "Finite-time Analysis of the Multiarmed Bandit Problem"
- Silver et al. (2016): "Mastering the game of Go with deep neural networks"

### Implementation References
- `hooks/log-feedback.sh`: Reward signal detection
- `scripts/confidence-calculator.sh`: Confidence score calculation
- `scripts/template-evolver.py`: Template evolution learning
- `docs/adr/001-reinforcement-learning-approach.md`: Design decisions

### Further Reading
- Multi-Armed Bandits: https://en.wikipedia.org/wiki/Multi-armed_bandit
- Q-Learning: https://en.wikipedia.org/wiki/Q-learning
- Exploration vs Exploitation: https://en.wikipedia.org/wiki/Exploration-exploitation_dilemma
