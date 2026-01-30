# Intent Parser

> **Component:** Predictive Context Engine  
> **Purpose:** Parse and classify user intent from natural language input  
> **Version:** 1.0.0

---

## Overview

The Intent Parser is the first component in the Predictive Context Engine. It analyzes user input to extract:
- **Explicit signals**: Keywords, commands, file references
- **Implicit signals**: Tone, urgency, context clues
- **Meta signals**: Session history patterns, time-of-day effects

---

## Intent Classification Taxonomy

### Primary Intent Types

| Type | Description | Typical Keywords |
|------|-------------|------------------|
| `DEBUG` | Fix bugs, resolve errors | bug, error, fix, broken, fail, crash |
| `DEVELOP` | Implement new features | create, add, build, implement, new |
| `REFACTOR` | Improve existing code | refactor, clean, optimize, improve |
| `REVIEW` | Assess code quality | review, check, PR, audit, inspect |
| `DOCUMENT` | Write/update docs | document, explain, describe, readme |
| `TEST` | Write or run tests | test, TDD, unit, integration, coverage |
| `DEPLOY` | Deployment operations | deploy, release, CI/CD, build, docker |
| `ANALYZE` | Understand code/system | analyze, understand, how, why, trace |
| `CONFIGURE` | Setup/configuration | setup, config, install, environment |
| `EXPLORE` | General exploration | what, where, find, search, list |

### Secondary Modifiers

| Modifier | Effect | Examples |
|----------|--------|----------|
| `URGENT` | Priority boost | ASAP, urgent, critical, production down |
| `CAREFUL` | Extra verification | carefully, make sure, double-check |
| `FAST` | Economy mode | quick, simple, just, only |
| `THOROUGH` | Premium mode | completely, fully, all cases |
| `LEARNING` | Include explanations | explain, teach, help me understand |

---

## Parsing Algorithm

### Step 1: Tokenization

```
Input: "Can you fix the login authentication bug? It's breaking production."

Tokens: [
  "fix" → signal: DEBUG
  "login" → domain: AUTH
  "authentication" → domain: AUTH
  "bug" → signal: DEBUG
  "breaking" → urgency: HIGH
  "production" → context: PRODUCTION, urgency: CRITICAL
]
```

### Step 2: Intent Scoring

```yaml
intent_scores:
  DEBUG: 0.85   # "fix" + "bug" + "breaking"
  DEVELOP: 0.10 # Could be new feature
  REFACTOR: 0.05 # Could be improvement

modifiers:
  URGENT: 0.95  # "production" + "breaking"
  domain: AUTH  # "login" + "authentication"
  
confidence: 0.85 # Primary intent confidence
```

### Step 3: Context Enrichment

```yaml
enriched_intent:
  primary: DEBUG
  confidence: 0.85
  
  domain: AUTH
  domain_confidence: 0.90
  
  urgency: CRITICAL
  urgency_signals: ["production", "breaking"]
  
  suggested_mode: CRITICAL  # 2x token budget
  
  keywords: ["fix", "login", "authentication", "bug"]
  
  context_hints:
    - "Check auth-related files recently modified"
    - "Look for authentication error logs"
    - "Consider security implications"
```

---

## Confidence Calculation

```
Base Confidence = (matched_keywords / total_keywords) * keyword_weight
                + (pattern_matches / known_patterns) * pattern_weight
                
Adjustments:
  + User history alignment: If user often does this type, +0.1
  + Session context alignment: If continues previous topic, +0.15
  + Explicit command: If user says "I want to debug", +0.25
  
  - Ambiguity: If multiple intents score similarly, -0.1 per overlap
  - Contradiction: If signals conflict, -0.2
  - Vagueness: If few signals, -0.15
```

---

## Disambiguation Strategies

When confidence < 0.6, apply these strategies:

### Strategy 1: Context Resolution
```
Check session history for recent similar intents.
If found: Boost matching intent by 0.15
```

### Strategy 2: Domain Resolution  
```
Identify domain from file paths, project type.
If domain matches an intent's typical domain: Boost by 0.10
```

### Strategy 3: Clarification Request
```
If still ambiguous, prepare clarifying question:
"I see you mentioned X and Y. Are you trying to:
 a) Debug an issue?
 b) Implement a new feature?
 c) Something else?"
```

---

## Output Schema

```yaml
intent_analysis:
  # Primary classification
  primary_intent: "DEBUG"
  primary_confidence: 0.85
  
  # Secondary classification
  secondary_intent: "DEVELOP"  # If applicable
  secondary_confidence: 0.15
  
  # Modifiers
  urgency: "critical|high|medium|low"
  mode_suggestion: "critical|premium|standard|economy"
  
  # Domain hints
  domain: "auth|api|frontend|database|..."
  domain_confidence: 0.90
  
  # Extracted signals
  keywords: ["fix", "login", "bug"]
  entities:
    files: ["auth.ts", "login.tsx"]
    functions: ["validateUser"]
    errors: ["AuthenticationError"]
  
  # Context hints for routing
  context_hints:
    suggested_files: ["src/auth/*", "tests/auth/*"]
    suggested_skills: ["debugging/systematic", "development/security"]
    risk_factors: ["production_impact", "security_sensitive"]
  
  # Disambiguation state
  needs_clarification: false
  clarification_prompt: null  # If needs_clarification is true
  
  # Metadata
  parse_duration_ms: 45
  tokens_analyzed: 12
```

---

## Integration Points

### Upstream
- Receives raw user input from Interface Layer

### Downstream
- Feeds into Pattern Predictor for context prediction
- Feeds into Skill Matcher for routing decisions
- Logs to Decision Audit Trail

---

## Performance Targets

| Metric | Target | Rationale |
|--------|--------|-----------|
| Parse latency | <100ms | User shouldn't wait for parsing |
| Confidence accuracy | >85% | Most predictions should be correct |
| Disambiguation rate | <20% | Most inputs should be clear |
| False positive rate | <5% | Wrong intent causes waste |

---

*Intent Parser v1.0.0 — Predictive Context Engine Component*
