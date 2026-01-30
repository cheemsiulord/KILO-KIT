# Decision Audit Trail (DAT)

> **Component:** Routing Engine  
> **Innovation:** #5 — Explainability System  
> **Purpose:** Log, trace, and explain all agent decisions  
> **Version:** 1.0.0

---

## Overview

The Decision Audit Trail (DAT) is Kilo-Kit's explainability system. Every decision made by the agent is logged with full reasoning context, enabling:

- **Debugging**: Understand why the agent behaved a certain way
- **Trust**: Users can verify decisions are reasonable  
- **Learning**: Historical decisions inform future improvements
- **Compliance**: Audit trail for sensitive operations

---

## Core Principles

### Principle 1: Nothing Hidden
```
Every decision, no matter how small, gets logged.
"Obvious" decisions often hide subtle bugs.
```

### Principle 2: Reasoning First
```
Don't just log WHAT was decided.
Log WHY it was decided that way.
Include alternatives considered and why rejected.
```

### Principle 3: Traceable Chain
```
Each decision links to its predecessors.
You can trace any outcome back to initial input.
```

### Principle 4: Queryable History
```
Logs aren't just for reading.
They're for analyzing patterns and anomalies.
```

---

## Decision Record Schema

```yaml
decision_record:
  # Unique identifier
  id: "DEC-{YYYYMMDD}-{HHMMSS}-{hash8}"
  timestamp: "2026-01-30T10:15:32.456Z"
  
  # Decision context
  context:
    session_id: "uuid"
    conversation_turn: 5
    user_input_hash: "sha256..."
    user_input_summary: "Fix login authentication bug..."
    token_budget_at_decision: 45000
    quality_mode: "critical"
  
  # Decision type
  decision_type: "routing|execution|quality_gate|error_recovery|completion"
  
  # The decision itself
  decision:
    action: "Route to skill"
    target: "skills/debugging/systematic"
    parameters:
      urgency: "critical"
      domain: "auth"
  
  # Reasoning chain (most important!)
  reasoning:
    # What signals led here
    signals:
      - source: "intent_parser"
        signal: "DEBUG intent detected"
        confidence: 0.85
        evidence: ["fix", "bug", "broken"]
      
      - source: "context"
        signal: "Production environment"
        confidence: 0.95
        evidence: ["production keyword"]
    
    # Candidates considered
    candidates:
      - option: "skills/debugging/systematic"
        score: 0.94
        pros: ["Matches DEBUG intent", "High success rate (87%)"]
        cons: ["Token heavy"]
      
      - option: "skills/debugging/quick-fix"
        score: 0.71
        pros: ["Token efficient"]
        cons: ["Lower success rate", "Production risk"]
      
      - option: "skills/quality/code-review"
        score: 0.45
        pros: []
        cons: ["Not a review request", "Wrong intent match"]
    
    # Selection rationale
    selection_rationale: >
      Selected 'systematic' over 'quick-fix' because:
      1. Production environment requires thoroughness
      2. User urgency is CRITICAL
      3. Auth domain has security implications
      4. 'systematic' has 87% success rate vs 62% for quick-fix
    
    # What wasn't chosen and why
    rejection_reasons:
      "quick-fix": "Production + security context requires thorough approach"
      "code-review": "Intent is DEBUG, not REVIEW"
  
  # Outcome (filled after execution)
  outcome:
    status: "pending|success|failure|partial"
    success_metrics:
      task_completed: true
      user_satisfied: null  # Filled from feedback
      tokens_used: 3200
      duration_ms: 45000
    failure_reason: null
    user_feedback: null
  
  # Learning updates triggered
  learning_impact:
    skill_score_delta: { "systematic": +0.02 }
    routing_weight_delta: { "DEBUG+AUTH→systematic": +0.05 }
    patterns_extracted: ["production_auth_bug→systematic_works"]
  
  # Links to related decisions
  related_decisions:
    triggered_by: "DEC-20260130-101530-abc12345"
    triggers: ["DEC-20260130-101545-def67890"]
```

---

## Decision Types

### Type 1: Routing Decisions
```yaml
decision_type: "routing"
description: "Which skill/behavior to use"
logged_when: "After intent parsing, before execution"
key_fields: [candidates, selection_rationale, rejection_reasons]
```

### Type 2: Execution Decisions
```yaml
decision_type: "execution"
description: "How to execute a behavior"
logged_when: "During behavior chain execution"
key_fields: [behavior, parameters, expected_outcome]
```

### Type 3: Quality Gate Decisions
```yaml
decision_type: "quality_gate"
description: "Pass or fail a checkpoint"
logged_when: "At each quality gate"
key_fields: [gate_name, criteria, result, evidence]
```

### Type 4: Error Recovery Decisions
```yaml
decision_type: "error_recovery"
description: "How to handle an error"
logged_when: "When error occurs during execution"
key_fields: [error, recovery_options, selected_recovery]
```

### Type 5: Completion Decisions
```yaml
decision_type: "completion"
description: "Whether to mark task complete"
logged_when: "Before claiming completion"
key_fields: [completion_criteria, verification_steps, verdict]
```

---

## Logging Protocol

### When to Log

```
LOG every:
  ✅ Skill/behavior selection
  ✅ Quality gate pass/fail
  ✅ Error occurrence and recovery
  ✅ User clarification request
  ✅ Completion claim
  ✅ Token budget decision
  ✅ Fallback activation

DON'T log:
  ❌ Intermediate computation steps (too verbose)
  ❌ File content (use references instead)
  ❌ Sensitive data (hash or omit)
```

### How to Log

```yaml
# Good logging
reasoning:
  selection_rationale: >
    Selected 'systematic' because production environment
    requires thorough investigation. Quick-fix rejected
    due to 23% failure rate on auth-related bugs.

# Bad logging (too vague)
reasoning:
  selection_rationale: "Seemed like the best option"

# Bad logging (missing alternatives)
reasoning:
  selection_rationale: "systematic is good for debugging"
  # Missing: what else was considered? why rejected?
```

---

## Query Patterns

### Pattern 1: Debug Agent Behavior
```
Query: "Why did agent choose X when user asked Y?"

Steps:
1. Find decision by timestamp or user_input_hash
2. Read reasoning.signals
3. Read reasoning.candidates
4. Read reasoning.selection_rationale
5. Read reasoning.rejection_reasons
```

### Pattern 2: Find Success Patterns
```
Query: "What decisions lead to successful outcomes?"

Steps:
1. Filter decisions where outcome.status == "success"
2. Group by decision_type and target
3. Extract common reasoning patterns
4. Identify high-confidence signals
```

### Pattern 3: Identify Failure Modes
```
Query: "What decisions lead to failures?"

Steps:
1. Filter decisions where outcome.status == "failure"
2. Analyze reasoning.selection_rationale
3. Check if alternatives would have worked
4. Identify corrective actions
```

### Pattern 4: User Trust Verification
```
Query: "Was this decision reasonable?"

Steps:
1. Review reasoning chain
2. Verify signals match input
3. Confirm alternatives were considered
4. Check outcome alignment with decision
```

---

## Storage Strategy

### Hot Storage (Current Session)
```
Location: In-memory
Retention: Session lifetime
Query speed: <1ms
```

### Warm Storage (Recent History)
```
Location: Local SQLite/JSON
Retention: 30 days
Query speed: <100ms
```

### Cold Storage (Archive)
```
Location: Compressed files
Retention: Optional (configurable)
Query speed: Seconds (requires decompression)
```

### Schema (SQLite Example)
```sql
CREATE TABLE decisions (
  id TEXT PRIMARY KEY,
  timestamp DATETIME,
  decision_type TEXT,
  target TEXT,
  outcome_status TEXT,
  reasoning_json TEXT,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_decisions_timestamp ON decisions(timestamp);
CREATE INDEX idx_decisions_type ON decisions(decision_type);
CREATE INDEX idx_decisions_outcome ON decisions(outcome_status);
```

---

## Privacy Considerations

### What Gets Hashed
- User input content (store hash, not original)
- File contents (store path + hash)
- Sensitive data (never store, only reference)

### What Gets Stored
- Decision metadata (always)
- Reasoning structure (always)
- Aggregated patterns (always)
- Anonymized statistics (always)

### Data Retention
- User controls retention period
- Default: 30 days rolling
- Anonymized aggregates: indefinite

---

## Integration Points

### Upstream
- Receives decisions from Routing Engine
- Receives decisions from Execution Engine
- Receives decisions from Quality Gates

### Downstream
- Feeds into Skill Analytics for learning
- Feeds into Debug/Trace tools
- Available for user inspection on request

---

*Decision Audit Trail v1.0.0 — Explainability System*
