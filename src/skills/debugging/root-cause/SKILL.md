---
name: root-cause-analysis
description: >-
  Deep root cause analysis using the 5 Whys and Fishbone techniques.
  Use when systematic debugging hasn't found the cause, or for complex systemic issues.
  Keywords: root cause, why, underlying, fundamental, systemic, deep, origin
version: 1.0.0
behaviors: [trace_error, investigate_codebase, reason]
dependencies: [debugging/systematic]
token_estimate:
  min: 2000
  typical: 4500
  max: 10000
---

# 🔬 Root Cause Analysis Skill

> **Philosophy:** Don't stop at the first "why" — dig until you hit bedrock.

## When to Use

Use this skill when:
- Systematic debugging found the bug but not WHY it exists
- Issue keeps recurring despite fixes
- Bug seems to have multiple contributing factors
- You need to prevent similar bugs in the future
- There's a systemic/architectural issue suspected

**Do NOT use this skill when:**
- Bug is simple and obvious
- Time is extremely limited (use quick-fix)
- Just need to patch, not understand

---

## Prerequisites

Before starting:
- [ ] Bug has been identified (what is happening)
- [ ] Have access to relevant code and history
- [ ] Understand the system architecture (high level)
- [ ] Have time for thorough analysis (~30-60 mins)

---

## Process

### Phase 1: PROBLEM DEFINITION 📝

**Goal:** Clearly define what we're analyzing.

**Steps:**

1. **State the Problem Precisely**
   ```
   Template:
   "When [condition], the system [actual behavior] 
    instead of [expected behavior]."
   
   Example:
   "When a user submits a login form with special characters,
    the system returns a 500 error instead of validating input."
   ```

2. **Gather Impact Data**
   - How often does it occur?
   - Who/what is affected?
   - What's the business impact?
   - How long has it been happening?

3. **Document Timeline**
   - When did it first appear?
   - Any recent changes before first occurrence?
   - Has it gotten better/worse?

**Output:** Clear problem statement with context.

---

### Phase 2: THE 5 WHYS ANALYSIS 🔍

**Goal:** Drill down to fundamental causes.

**Method:**

```
Start: Problem Statement
  │
  ├─ Why? → First-level cause
  │   │
  │   ├─ Why? → Second-level cause
  │   │   │
  │   │   ├─ Why? → Third-level cause
  │   │   │   │
  │   │   │   ├─ Why? → Fourth-level cause
  │   │   │   │   │
  │   │   │   │   └─ Why? → ROOT CAUSE
```

**Rules:**
1. Each answer must be factual, not speculative
2. If multiple answers possible at a level, branch and explore all
3. Stop when you reach something actionable
4. "Human error" is NEVER a root cause — dig deeper

**Example:**

```
Problem: Login fails with special characters

Why #1: Server returns 500 error
  → Because: Unhandled exception in auth.service.ts

Why #2: Why is there an unhandled exception?
  → Because: SQL query fails with syntax error

Why #3: Why does SQL have syntax error?
  → Because: User input is concatenated directly into query

Why #4: Why is input concatenated directly?
  → Because: Developer didn't use parameterized queries

Why #5: Why didn't developer use parameterized queries?
  → Because: No code review caught it, and no security guidelines exist

ROOT CAUSE: Missing security coding standards and review process
```

---

### Phase 3: FISHBONE DIAGRAM (ISHIKAWA) 📊

**Goal:** Explore contributing factors systematically.

**Categories to Examine:**

```
                              ┌──────────┐
          ┌──────────────────►│          │
          │    Environment    │          │
          │                   │          │
┌─────────┴───┐               │  PROBLEM │
│   Methods   ├──────────────►│          │
└─────────────┘               │          │
                              │          │
┌─────────────┐               │          │
│  Machines   ├──────────────►│          │
│  (Systems)  │               │          │
└─────────────┘               └─────┬────┘
                                    │
         ┌───────────────────────────┘
         │
    ┌────┴────┐    ┌──────────┐    ┌──────────┐
    │ People  │    │Materials │    │Measurement│
    │(Process)│    │  (Data)  │    │ (Metrics)│
    └─────────┘    └──────────┘    └──────────┘
```

**For Each Category, Ask:**

| Category | Questions to Ask |
|----------|------------------|
| **Methods** | Is the process correct? Is it followed? Is it documented? |
| **Machines** | Is the system configured correctly? Dependencies up to date? |
| **Environment** | Dev vs Prod differences? External factors? |
| **People/Process** | Training adequate? Communication clear? Handoffs smooth? |
| **Materials/Data** | Data quality? Input validation? Edge cases? |
| **Measurement** | Are we monitoring correctly? Are we measuring the right things? |

---

### Phase 4: CONTRIBUTING FACTOR ANALYSIS 🧩

**Goal:** Weight and prioritize contributing factors.

**Steps:**

1. **List All Contributing Factors**
   Combine findings from 5 Whys and Fishbone

2. **Score Each Factor**
   ```yaml
   scoring:
     frequency: 1-5 (how often does this contribute?)
     detectability: 1-5 (how hard to detect? 5=very hidden)
     severity: 1-5 (how much impact when it contributes?)
     
     risk_score: frequency × detectability × severity
   ```

3. **Create Priority Matrix**
   ```
   High Frequency + High Severity → Address immediately
   High Frequency + Low Severity → Address soon
   Low Frequency + High Severity → Create safeguards
   Low Frequency + Low Severity → Monitor only
   ```

---

### Phase 5: ROOT CAUSE VALIDATION ✅

**Goal:** Confirm the root cause is correct.

**Validation Questions:**

1. **Causation Test**
   - If we fix this, will the problem definitely not recur?
   - Can we prove cause → effect relationship?

2. **Completeness Test**
   - Are there other causes we might have missed?
   - Would fixing this alone be sufficient?

3. **Actionability Test**
   - Can we actually address this cause?
   - Is it within our control?

4. **Proportionality Test**
   - Is the root cause proportional to the problem?
   - (Big problems usually have big root causes)

---

### Phase 6: PREVENTION RECOMMENDATIONS 🛡️

**Goal:** Prevent recurrence.

**Recommendation Types:**

1. **Immediate Fix**
   - Direct fix for the symptom
   - Buys time for proper solution

2. **Root Cause Fix**
   - Addresses the fundamental cause
   - Prevents this exact issue

3. **Systemic Improvement**
   - Prevents entire class of similar issues
   - Usually involves process/tooling changes

4. **Detection Improvement**
   - Catch similar issues earlier next time
   - Monitoring, testing, review improvements

**Example Recommendations:**

```yaml
for_the_sql_injection_example:
  immediate:
    - Fix the specific query to use parameters
    - Add input sanitization
  
  root_cause:
    - Establish secure coding guidelines
    - Require security review for auth code
  
  systemic:
    - Enable SQL injection detection in SAST tooling
    - Add security-focused code review checklist
    - Security training for developers
  
  detection:
    - Add SQL injection tests to CI pipeline
    - Monitor for unusual database queries
```

---

## Output Template

```markdown
# Root Cause Analysis Report

## Problem Statement
[Clear statement of the problem]

## Timeline
- First observed: [date]
- Recent changes: [list]
- Frequency: [how often]

## 5 Whys Analysis
1. Why? → [answer]
2. Why? → [answer]
3. Why? → [answer]
4. Why? → [answer]
5. Why? → [answer]

## Contributing Factors
| Factor | Category | Risk Score | Priority |
|--------|----------|------------|----------|
| [factor] | [cat] | [score] | [priority] |

## Root Cause
[Clear statement of root cause]

## Validation
- [ ] Causation confirmed
- [ ] Complete (no other causes)
- [ ] Actionable
- [ ] Proportional

## Recommendations
### Immediate
- [action]

### Root Cause Fix
- [action]

### Systemic
- [action]

### Detection
- [action]
```

---

## Guidelines

### DO ✅
- Follow the evidence, not assumptions
- Keep asking "why" until you can't anymore
- Document everything for future reference
- Involve domain experts when needed
- Look for patterns across similar issues

### DON'T ❌
- Blame individuals (focus on systems)
- Stop at the first plausible answer
- Skip validation steps
- Propose fixes before understanding cause
- Ignore contributing factors

---

## Success Criteria

Before claiming analysis complete:

- [ ] Problem clearly defined with impact
- [ ] 5 Whys completed to true root cause
- [ ] Contributing factors identified and scored
- [ ] Root cause validated against all tests
- [ ] Prevention recommendations at all levels
- [ ] Report documented for future reference

---

## Related Skills

- `skills/debugging/systematic/` - For initial bug identification
- `skills/debugging/verification/` - For validating fixes
- `skills/quality/code-review/` - For review improvements

---

*Root Cause Analysis Skill v1.0.0 — Dig until you hit bedrock*
