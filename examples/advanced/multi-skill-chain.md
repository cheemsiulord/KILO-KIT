# Multi-Skill Chain: Complex Debugging Scenario

> **Difficulty:** Advanced  
> **Time:** 45 minutes  
> **Goal:** Chain multiple skills together for complex tasks

---

## Introduction

This tutorial demonstrates how to chain multiple Kilo-Kit skills together to handle a complex, real-world debugging scenario that no single skill could solve alone.

**Scenario:** A production bug that requires:
1. Systematic debugging to find the issue
2. Root cause analysis to understand why
3. Security review of the fix
4. Verification before deployment

---

## The Skill Chain

```
User Report: "Users can't login after password reset"
                          │
                          ▼
              ┌───────────────────────┐
              │ debugging/systematic  │
              │ • Investigate error   │
              │ • Locate bug          │
              └───────────┬───────────┘
                          │ Found: Token validation fails
                          ▼
              ┌───────────────────────┐
              │ debugging/root-cause  │
              │ • 5 Whys analysis     │
              │ • Find underlying cause│
              └───────────┬───────────┘
                          │ Root cause: Token expiry not refreshed
                          ▼
              ┌───────────────────────┐
              │ development/backend   │
              │ • Design the fix      │
              │ • Implement solution  │
              └───────────┬───────────┘
                          │ Fix implemented
                          ▼
              ┌───────────────────────┐
              │ development/security  │
              │ • Security review     │
              │ • Check for vulns     │
              └───────────┬───────────┘
                          │ Security approved
                          ▼
              ┌───────────────────────┐
              │ debugging/verification│
              │ • Test the fix        │
              │ • Verify no regression│
              └───────────┴───────────┘
                          │
                          ▼
                    Fix Deployed ✅
```

---

## Part 1: Setting Up the Chain

### Chain Definition

```yaml
# skill-chains/complex-debug.yaml
name: complex_production_debug
description: >-
  Full debugging pipeline for production issues
  requiring investigation, fix, and verification.

trigger:
  keywords: [production, critical, urgent, security, auth]
  confidence_threshold: 0.85

chain:
  skills:
    - id: investigate
      skill: debugging/systematic
      required: true
      output_as: investigation_result
    
    - id: root_cause
      skill: debugging/root-cause
      required: false  # Skip if obvious
      condition: "$investigation_result.confidence < 0.9"
      input_from: [investigate]
      output_as: root_cause_analysis
    
    - id: implement
      skill: development/backend
      required: true
      input_from: [investigate, root_cause]
      output_as: implementation
    
    - id: security_check
      skill: development/security
      required: true
      input_from: [implement]
      output_as: security_review
      on_fail: abort_chain
    
    - id: verify
      skill: debugging/verification
      required: true
      input_from: [implement, security_check]
      output_as: verification_result

  on_success:
    - action: generate_report
    - action: notify_team
  
  on_failure:
    - action: rollback
    - action: escalate
```

---

## Part 2: Skill Communication

### Passing Context Between Skills

Each skill produces output that flows to the next:

```yaml
# Investigation output
investigation_result:
  bug_found: true
  location:
    file: "src/auth/token.service.ts"
    lines: [45, 67]
  error_type: "TokenExpiredError"
  immediate_cause: "Token expiry timestamp not updated after password reset"
  confidence: 0.82

# Root cause output  
root_cause_analysis:
  root_cause: "Password reset flow doesn't refresh token expiry"
  contributing_factors:
    - "Token service lacks reset event listener"
    - "No integration test for password reset → login flow"
  prevention_recommendations:
    - "Add event listener for password reset"
    - "Add integration tests"

# Implementation output
implementation:
  files_changed:
    - file: "src/auth/token.service.ts"
      changes: ["Added password reset listener"]
    - file: "src/auth/events.ts"
      changes: ["Emit TOKEN_REFRESH on password reset"]
  tests_added:
    - "token.service.spec.ts: should refresh token on password reset"
  
# Security review output
security_review:
  passed: true
  issues_found: 0
  recommendations:
    - "Consider adding rate limiting to password reset"
  approved_for_production: true

# Verification output
verification_result:
  all_tests_pass: true
  regression_tests: "100% pass"
  manual_verification: "Login works after password reset"
  ready_for_deploy: true
```

---

## Part 3: Context Aggregation

### Building Cumulative Context

The chain maintains a growing context:

```yaml
chain_context:
  # Starts with user input
  initial:
    report: "Users can't login after password reset"
    urgency: "critical"
    affected_users: 150
  
  # After investigation
  after_investigate:
    $include: initial
    bug:
      location: "token.service.ts:45"
      cause: "Token expiry not refreshed"
    
  # After root cause
  after_root_cause:
    $include: after_investigate
    analysis:
      root_cause: "Missing event listener"
      5_whys_complete: true
  
  # After implementation
  after_implement:
    $include: after_root_cause
    fix:
      description: "Added password reset event handler"
      files_changed: 2
      tests_added: 1
  
  # After security
  after_security:
    $include: after_implement
    security:
      approved: true
      no_vulnerabilities: true
  
  # Final
  final:
    $include: after_security
    verification:
      passed: true
      ready: true
```

---

## Part 4: Error Handling in Chains

### Chain-Level Error Handling

```yaml
error_handling:
  # Per-skill error handling
  on_skill_error:
    investigate:
      retry: 2
      on_final_fail: abort
    
    root_cause:
      retry: 1
      on_final_fail: skip  # Not critical
    
    implement:
      retry: 1
      on_final_fail: abort
    
    security_check:
      retry: 0  # No retry - security is binary
      on_final_fail: abort_with_escalation
    
    verify:
      retry: 3
      on_final_fail: rollback

  # Chain-level handlers
  on_abort:
    actions:
      - log_failure
      - notify_on_call
      - create_incident
    
  on_rollback:
    actions:
      - git_revert
      - restore_backup
      - notify_team
```

### Conditional Skill Execution

```yaml
conditional_execution:
  # Skip root cause if issue is obvious
  root_cause:
    skip_if: "$investigation_result.confidence >= 0.95"
    reason: "High confidence means obvious issue"
  
  # Extra verification for security issues
  extra_security:
    skill: development/security
    run_if: "$investigation_result.tags contains 'security'"
    config:
      thoroughness: "maximum"
  
  # Regression testing for critical paths
  regression:
    skill: quality/testing
    run_if: "$implementation.affects_critical_path"
    config:
      scope: "full_regression"
```

---

## Part 5: Parallel Skill Execution

Some skills can run in parallel:

```yaml
parallel_execution:
  # These can run simultaneously
  parallel:
    - group: context_gathering
      skills:
        - investigate_code
        - investigate_logs
        - investigate_metrics
      merge_strategy: combine_all
    
    - group: verification
      skills:
        - unit_tests
        - integration_tests
        - e2e_tests
      merge_strategy: all_must_pass

  # These must be sequential
  sequential:
    - implement (needs investigation results)
    - security_check (needs implementation)
```

---

## Part 6: Live Example Walkthrough

### The Bug Report

```
Subject: CRITICAL - Users unable to login after password reset
Reported: 2026-01-30 14:30
Affected: ~150 users
Environment: Production

Steps to reproduce:
1. User resets password via "Forgot Password"
2. Sets new password successfully
3. Tries to login with new password
4. Gets "Session expired" error

Expected: User can login with new password
Actual: "Session expired" error on login
```

### Step 1: Systematic Investigation

**Skill:** `debugging/systematic`

```
🔍 Phase 1: Investigation

Analyzing error: "Session expired"
→ Checking token.service.ts
→ Found: validateToken() fails at line 45
→ Token.expiresAt is in the past despite fresh login

Analyzing timeline:
→ Password reset at 14:25
→ Login attempt at 14:28
→ Token expiry shows 14:20 (before password reset!)

Issue identified:
→ Token expiry is not refreshed when password changes
→ Old token expiry persists

Confidence: 0.82
```

### Step 2: Root Cause Analysis

**Skill:** `debugging/root-cause`

```
🔬 5 Whys Analysis

Why 1: Why does login fail?
→ Token validation fails (expiry in past)

Why 2: Why is expiry in past?
→ Token expiry wasn't updated during password reset

Why 3: Why wasn't it updated?
→ Password reset flow doesn't call token refresh

Why 4: Why doesn't it call token refresh?
→ Token service doesn't listen to password reset events

Why 5: Why doesn't it listen?
→ Missing integration between auth modules

ROOT CAUSE:
TokenService is not subscribed to PasswordResetEvent.
When password changes, tokens should be refreshed but this
connection was never implemented.
```

### Step 3: Implementation

**Skill:** `development/backend`

```
🔧 Fix Implementation

Design:
1. Subscribe TokenService to PasswordResetEvent
2. On event, refresh all user tokens
3. Add integration test

Implementation:
```typescript
// src/auth/token.service.ts
@Injectable()
export class TokenService {
  constructor(
    private eventBus: EventBus,
  ) {
    // NEW: Listen for password resets
    this.eventBus.subscribe(
      PasswordResetEvent,
      (event) => this.handlePasswordReset(event)
    );
  }
  
  private async handlePasswordReset(event: PasswordResetEvent) {
    // Refresh all tokens for this user
    await this.refreshUserTokens(event.userId);
  }
}
```

Test added:
```typescript
it('should refresh tokens on password reset', async () => {
  // Arrange
  const user = await createTestUser();
  const oldToken = await tokenService.createToken(user.id);
  
  // Act
  await authService.resetPassword(user.id, 'newPassword');
  
  // Assert
  const newToken = await tokenService.getActiveToken(user.id);
  expect(newToken.expiresAt).toBeAfter(oldToken.expiresAt);
});
```
```

### Step 4: Security Review

**Skill:** `development/security`

```
🔐 Security Review

Checking changes against OWASP Top 10:

✅ A01 Broken Access Control: No issues
✅ A02 Cryptographic Failures: No issues
✅ A07 Authentication Failures: ADDRESSED by this fix

Security checklist:
✅ Event handler validates userId
✅ Token refresh uses secure random
✅ Old tokens invalidated properly
✅ Audit log for token refresh
✅ Rate limiting on password reset (existing)

APPROVED for production.

Recommendations (non-blocking):
- Consider adding MFA for password reset
- Add monitoring for unusual token refresh patterns
```

### Step 5: Verification

**Skill:** `debugging/verification`

```
✅ Verification Complete

Direct verification:
✅ Reset password for test user
✅ Login successful with new password
✅ Token shows fresh expiry

Edge cases:
✅ Multiple password resets in sequence
✅ Reset during active session
✅ Reset with concurrent requests

Regression:
✅ All unit tests pass (156/156)
✅ All integration tests pass (43/43)
✅ Login flow e2e test pass

READY FOR DEPLOY
```

---

## Part 7: Chain Execution Report

```markdown
# Chain Execution Report

## Summary
- **Chain:** complex_production_debug
- **Started:** 2026-01-30 14:45:00
- **Completed:** 2026-01-30 15:32:00
- **Status:** ✅ SUCCESS

## Skills Executed
| Skill | Duration | Status | Tokens |
|-------|----------|--------|--------|
| debugging/systematic | 12m | ✅ | 3,200 |
| debugging/root-cause | 15m | ✅ | 4,100 |
| development/backend | 18m | ✅ | 5,500 |
| development/security | 8m | ✅ | 2,800 |
| debugging/verification | 14m | ✅ | 3,400 |

**Total time:** 47 minutes
**Total tokens:** 19,000

## Outcome
- Bug fixed: Token expiry not refreshed on password reset
- Root cause: Missing event subscription
- Security: Approved
- Tests: All pass

## Files Changed
1. `src/auth/token.service.ts` - Added event listener
2. `src/auth/token.service.spec.ts` - Added test

## Deployment
Ready for production deployment.
```

---

## Key Takeaways

1. **Chain skills for complex tasks** - No single skill can do everything
2. **Pass context between skills** - Build cumulative understanding
3. **Handle errors at chain level** - Unified error handling
4. **Use parallel when possible** - Speed up independent work
5. **Always verify at the end** - Chains should end with verification

---

## Related Tutorials

- [Behavior Composition](../intermediate/behavior-composition.md)
- [Custom Skill Creation](../intermediate/custom-workflow.md)
- [Token Optimization](../advanced/token-optimization.md)

---

*Multi-Skill Chain Tutorial v1.0.0*
