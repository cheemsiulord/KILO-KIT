# Skill Analytics (SET Implementation)

> **Component:** Knowledge Layer  
> **Innovation:** #2 — Skill Effectiveness Tracker  
> **Purpose:** Track, analyze, and improve skill performance  
> **Version:** 1.0.0

---

## Overview

Skill Analytics implements the **Skill Effectiveness Tracker (SET)** innovation. It collects metrics on every skill invocation, analyzes patterns, and provides data for adaptive routing and self-improvement.

**Key Insight:** Skills that don't improve over time become liabilities. SET ensures continuous improvement.

---

## Core Metrics

### Per-Skill Metrics

```yaml
skill_metrics:
  skill_id: "debugging/systematic"
  
  # Success Tracking
  invocation_count: 156
  success_count: 136
  failure_count: 20
  success_rate: 0.872        # success_count / invocation_count
  
  # Efficiency Tracking
  avg_token_usage: 3250
  token_efficiency: 0.72     # quality_per_token (normalized)
  avg_iterations: 2.3        # Average loops before completion
  
  # Time Tracking
  avg_duration_ms: 45000
  p50_duration_ms: 38000
  p90_duration_ms: 72000
  
  # User Satisfaction
  explicit_feedback_count: 23
  positive_feedback: 19
  negative_feedback: 4
  satisfaction_score: 0.826   # positive / total
  
  # Context Fit
  context_match_scores: [0.91, 0.88, 0.95, ...]
  avg_context_fit: 0.89
  
  # Trend Data
  last_30_days:
    success_rate: 0.89
    trend: "+0.02"           # Improving
  last_7_days:
    success_rate: 0.91
    trend: "+0.04"           # Improving faster
```

### Derived Metrics

```yaml
derived_metrics:
  # Overall Effectiveness Score (0.0 - 1.0)
  effectiveness_score: |
    0.4 × success_rate +
    0.2 × satisfaction_score +
    0.2 × token_efficiency +
    0.1 × (1 / normalized_iterations) +
    0.1 × avg_context_fit
  
  # Health Status
  health_status:
    excellent: effectiveness_score >= 0.85
    good: effectiveness_score >= 0.70
    needs_attention: effectiveness_score >= 0.50
    critical: effectiveness_score < 0.50
  
  # Routing Recommendation
  routing_modifier:
    excellent: +0.10          # Boost in skill matching
    good: 0                   # No modification
    needs_attention: -0.10    # Slight penalty
    critical: -0.25           # Significant penalty
```

---

## Data Collection

### On Every Invocation

```yaml
invocation_record:
  timestamp: "2026-01-30T10:15:32Z"
  skill_id: "debugging/systematic"
  session_id: "uuid"
  
  # Input Context
  input:
    intent_type: "DEBUG"
    intent_confidence: 0.85
    keywords: ["fix", "login", "bug"]
    context_fit_score: 0.91
  
  # Execution Data
  execution:
    behaviors_executed: ["parse_error", "search_code", "reason", "write_file"]
    iterations: 2
    token_usage: 3100
    duration_ms: 42000
    quality_gates_passed: ["gate1", "gate2", "gate3", "gate4"]
    quality_gates_failed: []
  
  # Outcome
  outcome:
    success: true
    error_if_any: null
    user_feedback: "positive"  # positive | negative | neutral | none
    
  # Learning Extracted
  patterns:
    - "auth_login_bug → systematic → success"
    - "typescript + auth → avg 2 iterations"
```

### Feedback Collection

```yaml
feedback_sources:
  explicit:
    - User says "thanks" / "great" → positive
    - User says "wrong" / "that's not it" → negative
    - No comment → neutral
  
  implicit:
    - User proceeds with output → positive signal
    - User asks to redo → negative signal
    - User abandons task → negative signal
    - User asks follow-up → neutral
  
  system:
    - All quality gates pass → positive
    - Gates fail and recover → neutral
    - Gates fail, cannot recover → negative
```

---

## Analysis & Insights

### Pattern Extraction

```yaml
pattern_analysis:
  common_success_patterns:
    - pattern: "DEBUG + AUTH → systematic"
      success_rate: 0.92
      recommendation: "Route AUTH debugging to systematic first"
    
    - pattern: "3rd+ attempt → root-cause"
      success_rate: 0.88
      recommendation: "Switch to root-cause after 2 failures"
  
  common_failure_patterns:
    - pattern: "complex + economy_mode"
      failure_rate: 0.45
      recommendation: "Don't use economy mode for complex tasks"
    
    - pattern: "async + systematic"
      iteration_increase: +1.5
      recommendation: "Consider async-specific debugging skill"
  
  skill_chains:
    - chain: ["code-review", "debugging/systematic"]
      success_rate: 0.95
      recommendation: "Review before debug is effective"
    
    - chain: ["debugging/systematic", "debugging/verification"]
      success_rate: 0.98
      recommendation: "Always verify after systematic"
```

### Skill Improvement Suggestions

```yaml
improvement_suggestions:
  skill: "debugging/systematic"
  current_effectiveness: 0.78
  
  suggestions:
    - issue: "High iteration count for async bugs"
      suggestion: "Add async-specific troubleshooting section"
      expected_improvement: "+0.05 effectiveness"
    
    - issue: "Low success rate for database issues"
      suggestion: "Add database debugging patterns or defer to specialized skill"
      expected_improvement: "+0.08 effectiveness"
    
    - issue: "Token usage higher than similar skills"
      suggestion: "Optimize context loading, consider lazy loading references"
      expected_improvement: "+10% token efficiency"
```

---

## Self-Improvement Loop

```
┌────────────────────────────────────────────────────────────┐
│              SKILL EFFECTIVENESS TRACKER                    │
├────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐ │
│  │ COLLECT │───►│ ANALYZE │───►│ SUGGEST │───►│ IMPROVE │ │
│  │  Data   │    │ Patterns│    │ Changes │    │  Skill  │ │
│  └─────────┘    └─────────┘    └─────────┘    └─────────┘ │
│       ▲                                            │       │
│       └────────────────────────────────────────────┘       │
│                                                             │
│  Automated Flow:                                            │
│  1. Collect metrics on every invocation                    │
│  2. Analyze patterns weekly                                │
│  3. Generate improvement suggestions                       │
│  4. Flag skills needing attention                          │
│  5. Track improvement over time                            │
└────────────────────────────────────────────────────────────┘
```

### Improvement Actions

```yaml
improvement_actions:
  automated:
    - Adjust routing weights based on success rates
    - Flag underperforming skills for review
    - Suggest skill chains based on patterns
    
  semi_automated:
    - Generate improvement suggestions
    - Create skill comparison reports
    - Identify gaps in skill coverage
    
  manual:
    - Rewrite skill sections based on feedback
    - Create new skills for uncovered patterns
    - Deprecate ineffective skills
```

---

## Reporting

### Skill Report (Per-Skill)

```yaml
skill_report:
  skill_id: "debugging/systematic"
  period: "last_30_days"
  generated: "2026-01-30T18:00:00Z"
  
  summary:
    effectiveness_score: 0.82
    health_status: "good"
    trend: "improving"
    
  key_metrics:
    invocations: 45
    success_rate: 0.87
    avg_iterations: 2.1
    avg_tokens: 3100
    satisfaction: 0.85
    
  strengths:
    - "High success rate for TypeScript projects (0.92)"
    - "Good at auth-related bugs (0.89)"
    - "Users often follow up with 'thanks'"
    
  weaknesses:
    - "Struggles with async/timing issues (0.71)"
    - "High token usage compared to quick-fix alternative"
    
  recommendations:
    - "Add async troubleshooting section"
    - "Consider lazy-loading reference docs"
    
  compared_to_similar:
    vs_quick_fix: "+15% success, +40% tokens"
    vs_root_cause: "-5% success, faster"
```

### Dashboard Summary

```yaml
dashboard:
  total_skills: 25
  
  by_health:
    excellent: 8
    good: 12
    needs_attention: 4
    critical: 1
    
  top_performers:
    - "debugging/verification" (0.95)
    - "quality/code-review" (0.92)
    - "development/security" (0.90)
    
  needs_improvement:
    - "architecture/scalability" (0.58)
    - "automation/workflows" (0.62)
    
  recent_trends:
    improving: 15 skills
    stable: 8 skills
    declining: 2 skills
    
  coverage_gaps:
    - "No skill for mobile debugging"
    - "Low coverage for GraphQL issues"
```

---

## Storage Schema

```sql
-- Skill Metrics Table
CREATE TABLE skill_metrics (
  skill_id TEXT PRIMARY KEY,
  invocation_count INTEGER,
  success_count INTEGER,
  failure_count INTEGER,
  total_tokens BIGINT,
  total_duration_ms BIGINT,
  positive_feedback INTEGER,
  negative_feedback INTEGER,
  last_updated TIMESTAMP
);

-- Invocation Records Table
CREATE TABLE invocation_records (
  id TEXT PRIMARY KEY,
  skill_id TEXT,
  timestamp TIMESTAMP,
  success BOOLEAN,
  token_usage INTEGER,
  duration_ms INTEGER,
  iterations INTEGER,
  context_fit_score REAL,
  user_feedback TEXT,
  patterns_json TEXT
);

-- Pattern Analysis Table
CREATE TABLE patterns (
  id TEXT PRIMARY KEY,
  pattern TEXT,
  skill_id TEXT,
  success_rate REAL,
  occurrence_count INTEGER,
  recommendation TEXT,
  last_seen TIMESTAMP
);

-- Indexes for efficient queries
CREATE INDEX idx_invocations_skill ON invocation_records(skill_id);
CREATE INDEX idx_invocations_time ON invocation_records(timestamp);
CREATE INDEX idx_patterns_skill ON patterns(skill_id);
```

---

## Configuration

```yaml
skill_analytics_config:
  # Collection settings
  collect_metrics: true
  collect_patterns: true
  feedback_sampling_rate: 1.0    # 100% of invocations
  
  # Analysis settings
  analysis_frequency: "daily"
  pattern_min_occurrences: 5
  trend_window_days: 30
  
  # Alerting
  alert_on_critical: true
  alert_threshold: 0.50
  
  # Retention
  raw_records_retention_days: 90
  aggregated_metrics_retention: "indefinite"
  
  # Reporting
  report_frequency: "weekly"
  dashboard_refresh: "hourly"
```

---

## Integration Points

### Upstream
- Receives execution results from Execution Engine
- Receives feedback signals from user interaction

### Downstream
- Provides routing modifiers to Skill Matcher
- Provides reports to Dashboard/CLI
- Suggests improvements to skill maintainers

---

*Skill Analytics v1.0.0 — Self-improving skill system*
