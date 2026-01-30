# Token Economy Manager (TEM)

> **Component:** Execution Engine  
> **Innovation:** #4 — Cost/Quality Optimization  
> **Purpose:** Manage token budgets with intelligent quality/cost tradeoffs  
> **Version:** 1.0.0

---

## Overview

The Token Economy Manager (TEM) ensures that Kilo-Kit operates efficiently within resource constraints while maintaining quality standards. It treats tokens as a finite resource that must be budgeted, tracked, and optimized.

**Key Advantage:** Unlike traditional reactive systems that operate without cost awareness, TEM proactively manages resources to prevent waste on simple tasks and ensure sufficient context for complex ones.

---

## Core Concepts

### Token Budget

```yaml
token_budget:
  daily_limit: 100000      # User-configured
  session_limit: 50000     # Per conversation session
  task_limit: null         # Per-task (optional)
  
  current_state:
    daily_used: 45230
    daily_remaining: 54770
    session_used: 12450
    session_remaining: 37550
```

### Quality Modes

| Mode | Multiplier | Behavior | Use Case |
|------|------------|----------|----------|
| 🟢 **Economy** | 0.6x | Minimal exploration, direct answers | Simple lookups, quick fixes |
| 🟡 **Standard** | 1.0x | Balanced approach | Most tasks |
| 🟠 **Premium** | 1.5x | Deeper analysis, more verification | Complex tasks, unfamiliar code |
| 🔴 **Critical** | 2.0x | Maximum thoroughness, multiple passes | Production issues, security |

### Token Cost Categories

```yaml
token_costs:
  context_loading:
    skill_metadata: 50-100
    skill_body: 500-2000
    references: 1000-5000
    code_context: 100-500 per file
  
  reasoning:
    simple_decision: 100-300
    complex_analysis: 500-1500
    deep_reasoning: 2000-5000
  
  generation:
    short_response: 100-300
    code_block: 200-1000
    full_implementation: 1000-5000
  
  verification:
    syntax_check: 50-100
    test_run_analysis: 200-500
    comprehensive_review: 500-1500
```

---

## Budget Management Protocol

### Pre-Task Estimation

Before starting any task, TEM estimates token requirements:

```yaml
estimate_request:
  task_type: "DEBUG"
  domain: "AUTH"
  complexity_signals:
    files_involved: 3
    estimated_reasoning_depth: "medium"
    verification_needed: true
    
estimate_response:
  context_tokens: 2500
    breakdown:
      skill_loading: 1500
      code_context: 1000
  
  reasoning_tokens: 1800
    breakdown:
      analysis: 1000
      hypothesis: 500
      decision: 300
  
  generation_tokens: 2000
    breakdown:
      fix_code: 1500
      explanation: 500
  
  verification_tokens: 700
    breakdown:
      syntax_check: 200
      test_analysis: 500
  
  total_estimated: 7000
  confidence: 0.75
  recommended_mode: "standard"
  
  budget_check:
    sufficient: true
    remaining_after: 30550
    warning: null
```

### Real-Time Tracking

During execution, TEM tracks actual usage:

```yaml
tracking:
  task_id: "TASK-001"
  
  budget:
    allocated: 7000
    used: 4250
    remaining: 2750
    
  by_category:
    context: { allocated: 2500, used: 2100 }
    reasoning: { allocated: 1800, used: 1400 }
    generation: { allocated: 2000, used: 750 }
    verification: { allocated: 700, used: 0 }
    
  status: "on_track"  # on_track | warning | overspent
  adjustments_made: 0
```

### Budget Alerts

```yaml
alert_thresholds:
  warning: 0.8   # 80% of allocation used
  critical: 0.95 # 95% of allocation used
  
alert_actions:
  warning:
    - Log to decision trail
    - Consider switching to Economy mode
    - Evaluate remaining work vs budget
  
  critical:
    - Force mode switch to Economy
    - Prioritize completion over thoroughness
    - Notify user of constraint
  
  overspent:
    - Complete current operation
    - Summarize remaining work
    - Request user decision on continuation
```

---

## Mode Selection Algorithm

```python
def select_quality_mode(intent, context, budget):
    """
    Algorithm to select appropriate quality mode.
    """
    
    # Force Critical for specific scenarios
    if context.is_production and intent.type == "DEBUG":
        return Mode.CRITICAL
    
    if intent.domain == "SECURITY":
        return Mode.CRITICAL
    
    # Force Economy for simple scenarios
    if intent.confidence > 0.95 and context.task_familiarity > 0.9:
        return Mode.ECONOMY
    
    if intent.modifiers.contains("quick") or intent.modifiers.contains("simple"):
        return Mode.ECONOMY
    
    # Premium for complex scenarios
    if context.complexity > 7:
        return Mode.PREMIUM
    
    if context.files_touched > 5:
        return Mode.PREMIUM
    
    if intent.confidence < 0.6:  # Uncertain, need exploration
        return Mode.PREMIUM
    
    # Budget-aware downgrade
    if budget.remaining < budget.estimated * 1.5:
        if current_mode == Mode.CRITICAL:
            return Mode.PREMIUM
        if current_mode == Mode.PREMIUM:
            return Mode.STANDARD
        # Never auto-downgrade to Economy from Standard
    
    # Default
    return Mode.STANDARD
```

---

## Quality/Cost Tradeoffs

### Economy Mode Behavior

```yaml
economy_mode:
  context_loading:
    load_skill_metadata: yes
    load_skill_body: minimal_sections_only
    load_references: no
    load_code_context: current_file_only
  
  reasoning:
    hypothesis_count: 1
    verification_depth: "syntax_only"
    alternative_exploration: no
  
  generation:
    explanation_detail: "minimal"
    code_comments: "essential_only"
    
  verification:
    run_tests: no
    comprehensive_review: no
```

### Critical Mode Behavior

```yaml
critical_mode:
  context_loading:
    load_skill_metadata: yes
    load_skill_body: complete
    load_references: all_relevant
    load_code_context: related_files_plus_dependencies
  
  reasoning:
    hypothesis_count: 3
    verification_depth: "comprehensive"
    alternative_exploration: yes
    second_opinion: yes  # Re-analyze with fresh perspective
  
  generation:
    explanation_detail: "comprehensive"
    code_comments: "thorough"
    include_edge_cases: yes
    
  verification:
    run_tests: yes
    comprehensive_review: yes
    security_check: yes
    performance_check: if_applicable
```

---

## Optimization Strategies

### Strategy 1: Context Caching

```yaml
caching:
  description: "Reuse loaded context across related tasks"
  
  cache_policy:
    skill_metadata: cache_indefinitely
    skill_body: cache_session
    code_context: cache_until_modified
    
  savings: "30-50% on related sequential tasks"
```

### Strategy 2: Progressive Disclosure

```yaml
progressive_disclosure:
  description: "Load context in stages, stop when sufficient"
  
  stages:
    1: skill_metadata  # Usually enough to route
    2: skill_body      # Needed for execution
    3: references      # Only if body insufficient
    
  savings: "40-60% when early stages suffice"
```

### Strategy 3: Batch Optimization

```yaml
batch_optimization:
  description: "Optimize similar sequential tasks"
  
  detection:
    - Same skill used 3+ times consecutively
    - Similar file patterns
    - Related operations
  
  optimization:
    - Keep skill loaded
    - Batch context loading
    - Parallelize independent operations
    
  savings: "25-40% on batch operations"
```

### Strategy 4: Lazy Generation

```yaml
lazy_generation:
  description: "Generate only what's needed"
  
  triggers:
    - User explicitly requests detail
    - Verification fails
    - Complexity exceeds threshold
  
  default_behavior:
    - Concise responses
    - Expandable sections
    - Details on request
    
  savings: "20-30% on typical tasks"
```

---

## Reporting

### Per-Task Report

```yaml
task_report:
  task_id: "TASK-001"
  duration: "45s"
  
  tokens:
    estimated: 7000
    actual: 6200
    efficiency: 0.89  # actual/estimated (lower is better for under-budget)
    
  by_phase:
    context: { estimated: 2500, actual: 2100, delta: -400 }
    reasoning: { estimated: 1800, actual: 1600, delta: -200 }
    generation: { estimated: 2000, actual: 1800, delta: -200 }
    verification: { estimated: 700, actual: 700, delta: 0 }
  
  mode_used: "standard"
  mode_adjustments: 0
  
  optimization_opportunities:
    - "Context for auth.ts was loaded twice (could cache)"
    - "Verification ran full suite when unit tests would suffice"
```

### Daily Summary

```yaml
daily_summary:
  date: "2026-01-30"
  
  budget:
    limit: 100000
    used: 67500
    remaining: 32500
    utilization: 0.675
  
  tasks:
    completed: 23
    avg_tokens: 2935
    
  by_mode:
    economy: { count: 8, tokens: 9600, avg: 1200 }
    standard: { count: 12, tokens: 36000, avg: 3000 }
    premium: { count: 2, tokens: 15000, avg: 7500 }
    critical: { count: 1, tokens: 6900, avg: 6900 }
  
  efficiency_score: 0.82  # Actual vs estimated across all tasks
  
  recommendations:
    - "Consider Economy mode for lookup tasks (used Standard 5 times)"
    - "Critical mode task completed under budget (could use Premium next time)"
```

---

## Integration Points

### Upstream
- Receives task estimates from Intent Parser
- Receives mode recommendations from Routing Engine

### Downstream
- Controls Execution Engine behavior
- Reports to Decision Audit Trail
- Feeds into Skill Analytics

---

## Configuration

```yaml
# User-configurable settings
token_economy_config:
  daily_budget: 100000
  session_budget: 50000
  
  default_mode: "standard"
  
  auto_mode_switch: true  # Allow automatic mode adjustments
  
  warning_threshold: 0.8
  critical_threshold: 0.95
  
  report_frequency: "per_task"  # per_task | hourly | daily
  
  optimization:
    caching_enabled: true
    batch_detection: true
    lazy_generation: true
```

---

*Token Economy Manager v1.0.0 — Cost/Quality Optimization System*
