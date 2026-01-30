---
name: code-review
description: >-
  Comprehensive code review checklist and methodology.
  Use when reviewing PRs, conducting code audits, or assessing code quality.
  Keywords: review, PR, code review, audit, assess, quality, check
version: 1.0.0
behaviors: [read_file, reason, validate]
dependencies: []
token_estimate:
  min: 1000
  typical: 2500
  max: 5000
---

# 👁️ Code Review Skill

> **Philosophy:** Code review is collaboration, not criticism.

## When to Use

Use this skill when:
- Reviewing a Pull Request
- Conducting a code audit
- Assessing code quality before merge
- Mentoring through code feedback
- Preparing code for production

**Do NOT use this skill when:**
- Just need to run linter
- Simple typo fix
- Automated formatting changes only

---

## Prerequisites

Before starting review:
- [ ] Understand the purpose/goal of the change
- [ ] Have context on the project architecture
- [ ] Know the coding standards for the project
- [ ] Can run the code locally (if needed)

---

## Process

### Phase 1: CONTEXT UNDERSTANDING 📋

**Goal:** Understand WHAT and WHY before HOW.

**Steps:**

1. **Read the PR Description**
   - What problem does this solve?
   - What approach was taken?
   - Are there any caveats noted?

2. **Check Related Issues**
   - Link to issue/ticket
   - Requirements met?
   - Edge cases addressed?

3. **Assess Scope**
   - How many files changed?
   - Is this focused or sprawling?
   - Should this be multiple PRs?

**Output:** Clear understanding of change purpose.

---

### Phase 2: HIGH-LEVEL REVIEW 🔭

**Goal:** Evaluate architecture and design decisions.

**Checklist:**

```
DESIGN
□ Does the solution make sense?
□ Is this the right place for this code?
□ Does it follow project patterns?
□ Is it over-engineered?
□ Is it under-engineered?

ARCHITECTURE
□ Proper separation of concerns?
□ Dependencies going the right direction?
□ New dependencies justified?
□ Breaking any architectural boundaries?

SCOPE
□ Does change match stated purpose?
□ Any scope creep?
□ Any missing pieces?
```

**Output:** Assessment of overall approach.

---

### Phase 3: LINE-BY-LINE REVIEW 🔍

**Goal:** Examine code quality and correctness.

**Checklist:**

```
CORRECTNESS
□ Logic is correct
□ Edge cases handled
□ Error cases handled
□ Null/undefined handled
□ No off-by-one errors
□ Concurrency issues addressed

QUALITY
□ Clear variable/function names
□ Single responsibility principle
□ DRY (no unnecessary duplication)
□ Comments explain WHY, not WHAT
□ No dead code
□ No commented-out code
□ No TODOs without tracking

SECURITY
□ Input validation
□ No SQL injection risks
□ No XSS risks
□ Secrets not hardcoded
□ Proper authentication checks
□ Authorization verified

PERFORMANCE
□ No obvious N+1 queries
□ Appropriate caching
□ No blocking operations where async needed
□ Large data sets handled efficiently
```

**Output:** Detailed feedback on code quality.

---

### Phase 4: TESTING REVIEW 🧪

**Goal:** Ensure adequate test coverage.

**Checklist:**

```
TEST PRESENCE
□ Tests added for new functionality?
□ Tests updated for modified functionality?
□ Test file naming consistent?

TEST QUALITY
□ Tests are meaningful (not just coverage)?
□ Edge cases tested?
□ Error cases tested?
□ Tests are independent/isolated?
□ No flaky tests introduced?

TEST COVERAGE
□ Happy path covered?
□ Unhappy path covered?
□ Boundary conditions covered?
```

**Output:** Assessment of test adequacy.

---

### Phase 5: FINAL CHECKS ✅

**Goal:** Ensure readiness for merge.

**Checklist:**

```
DOCUMENTATION
□ README updated if needed?
□ API docs updated if needed?
□ Inline comments sufficient?
□ Migration guide if breaking changes?

OPERATIONAL
□ Logs added for debugging?
□ Metrics/monitoring considered?
□ Feature flags if needed?
□ Rollback plan if needed?

MERGE READINESS
□ CI passes?
□ No merge conflicts?
□ Approved by required reviewers?
□ All conversations resolved?
```

**Output:** Clear approve/request changes decision.

---

## Review Comment Guidelines

### Categorize Your Comments

| Prefix | Meaning | Action Required |
|--------|---------|-----------------|
| `🔴 BLOCKER:` | Must fix before merge | Yes, mandatory |
| `🟡 SUGGESTION:` | Should consider | Recommended |
| `🟢 NIT:` | Minor, optional | No |
| `❓ QUESTION:` | Need clarification | Response needed |
| `💡 IDEA:` | Future improvement | No |
| `👍 PRAISE:` | Great work! | No |

### Example Comments

**Good:**
```
🔴 BLOCKER: This SQL query is vulnerable to injection.
Use parameterized queries instead:
`db.query("SELECT * FROM users WHERE id = ?", [userId])`
```

**Bad:**
```
This is wrong.
```

### Tone Guidelines

- ✅ "Consider using X because Y"
- ✅ "What happens if Z is null?"
- ✅ "Great use of pattern X!"
- ❌ "This is stupid"
- ❌ "Obviously you should..."
- ❌ "Why didn't you just..."

---

## Common Issues to Watch For

### Security Issues

| Issue | Detection | Solution |
|-------|-----------|----------|
| SQL Injection | String concatenation in queries | Parameterized queries |
| XSS | Unescaped user input in HTML | Proper escaping/encoding |
| Hardcoded secrets | API keys in code | Environment variables |
| Missing auth | Endpoints without checks | Add auth middleware |

### Performance Issues

| Issue | Detection | Solution |
|-------|-----------|----------|
| N+1 queries | Loop with DB calls | Batch/eager loading |
| Missing index | Slow queries on large tables | Add database index |
| Blocking I/O | Sync calls in async context | Use async/await |
| Memory leak | Unbounded caches/listeners | Cleanup/limits |

### Code Quality Issues

| Issue | Detection | Solution |
|-------|-----------|----------|
| God function | 100+ lines, many responsibilities | Break into smaller functions |
| Magic numbers | `if (status === 3)` | Named constants |
| Deep nesting | 4+ levels of if/for | Early returns, extraction |
| Copy-paste code | Similar blocks repeated | Extract utility function |

---

## Success Criteria

Before approving:

- [ ] I understand what this code does and why
- [ ] The approach is appropriate for the problem
- [ ] Code is correct and handles edge cases
- [ ] Code is secure (no obvious vulnerabilities)
- [ ] Tests are adequate and meaningful
- [ ] Code is readable and maintainable
- [ ] No blocking issues remain
- [ ] All my questions have been answered

---

## Related Skills

- `skills/quality/testing/` - For test quality guidance
- `skills/development/security/` - For security review
- `skills/debugging/systematic/` - If bugs found during review

---

*Code Review Skill v1.0.0 — Collaboration, not criticism*
