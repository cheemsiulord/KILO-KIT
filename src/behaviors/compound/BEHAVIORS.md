# Compound Behaviors Reference

> **Component:** Behavior Layer  
> **Type:** Compound (Composed from atomic behaviors)  
> **Version:** 1.0.0

---

## Overview

Compound behaviors are higher-level actions composed from multiple atomic behaviors. They represent common workflows that appear frequently across skills.

**Properties of Compound Behaviors:**
- Built from 2+ atomic behaviors
- Represent complete micro-workflows
- Reusable across multiple skills
- Have defined input/output contracts
- Can be further composed into meta-behaviors

---

## Catalog of Compound Behaviors

### 1. `trace_error`

**Purpose:** Trace an error from symptom to root cause.

```yaml
behavior: trace_error
category: debugging
composed_of: [parse_input, search_code, read_file, reason]

input:
  error_message: string
  stack_trace: string (optional)
  context_files: array<string> (optional)

output:
  root_cause: string
  affected_files: array<{file, lines}>
  reasoning_chain: array<{step, finding}>
  confidence: float
  tokens_used: int

token_cost: ~400-800

workflow:
  1. parse_input(error_message) → parsed_error
  2. search_code(parsed_error.keywords) → potential_locations
  3. parallel:
     - read_file(location) for each location
  4. reason(all_context) → root_cause_analysis

example:
  input:
    error_message: "TypeError: Cannot read property 'id' of undefined"
    stack_trace: "at UserService.getUser (user.service.ts:45)"
  output:
    root_cause: "User object is null when accessed before database fetch completes"
    affected_files:
      - file: "user.service.ts"
        lines: [42, 45, 48]
    confidence: 0.87
```

---

### 2. `modify_safely`

**Purpose:** Safely modify a file with validation and backup.

```yaml
behavior: modify_safely
category: mutation
composed_of: [read_file, validate_syntax, write_file, run_command]

input:
  file_path: string
  modification: string | object
  modification_type: "replace" | "insert" | "append"
  at_location: string | int (optional)
  run_tests_after: bool (default: true)

output:
  success: bool
  backup_path: string
  validation_result: object
  test_result: object (if run_tests_after)
  tokens_used: int

token_cost: ~200-400

workflow:
  1. read_file(file_path) → original_content
  2. apply_modification(original_content, modification) → new_content
  3. validate_syntax(new_content) → validation
  4. if validation.valid:
       write_file(file_path, new_content, backup=true)
       if run_tests_after:
         run_command(test_command)
     else:
       return validation_errors

example:
  input:
    file_path: "src/auth.ts"
    modification: "if (!user) return null;"
    modification_type: "insert"
    at_location: 45
  output:
    success: true
    backup_path: "src/auth.ts.backup.1706612345"
    validation_result: { valid: true }
    test_result: { passed: true, tests_run: 5 }
```

---

### 3. `test_change`

**Purpose:** Write a change and verify it with tests.

```yaml
behavior: test_change
category: quality
composed_of: [write_file, run_command, parse_input, compare]

input:
  file_path: string
  new_content: string
  test_command: string
  expected_outcome: "pass" | "fail" | "specific_output"
  expected_output: string (if specific_output)

output:
  change_applied: bool
  test_passed: bool
  test_output: string
  matches_expected: bool
  tokens_used: int

token_cost: ~150-300

workflow:
  1. write_file(file_path, new_content)
  2. run_command(test_command) → test_result
  3. parse_input(test_result.stdout) → parsed_result
  4. compare(parsed_result, expected) → match

example:
  input:
    file_path: "src/auth.ts"
    new_content: "function validateUser(user) { return user?.id != null; }"
    test_command: "npm test -- --grep 'validateUser'"
    expected_outcome: "pass"
  output:
    change_applied: true
    test_passed: true
    test_output: "2 passing (45ms)"
    matches_expected: true
```

---

### 4. `investigate_codebase`

**Purpose:** Thoroughly investigate a part of the codebase.

```yaml
behavior: investigate_codebase
category: analysis
composed_of: [search_code, read_file, summarize, reason]

input:
  topic: string
  entry_points: array<string> (optional)
  depth: "shallow" | "medium" | "deep"
  max_files: int (default: 10)

output:
  summary: string
  key_files: array<{file, purpose, importance}>
  dependencies: array<{from, to, type}>
  patterns_found: array<string>
  recommendations: array<string>
  tokens_used: int

token_cost: ~500-1500

workflow:
  1. search_code(topic) → initial_matches
  2. parallel:
     - read_file(file) for top matches
  3. for each file:
     - search_code(imported_modules) → dependencies
  4. summarize(all_findings) → summary
  5. reason(summary) → recommendations

example:
  input:
    topic: "authentication"
    depth: "medium"
  output:
    summary: "Authentication is handled by UserService with JWT tokens..."
    key_files:
      - file: "auth.service.ts"
        purpose: "Main authentication logic"
        importance: "critical"
    dependencies:
      - from: "auth.controller.ts"
        to: "auth.service.ts"
        type: "import"
    patterns_found: ["JWT authentication", "Middleware pattern"]
    recommendations: ["Add rate limiting", "Consider refresh tokens"]
```

---

### 5. `generate_with_validation`

**Purpose:** Generate code/content with validation loop.

```yaml
behavior: generate_with_validation
category: generation
composed_of: [generate, validate_syntax, validate_data, reason]

input:
  specification: string
  type: "code" | "config" | "documentation"
  language: string (if code)
  validation_rules: array<string>
  max_attempts: int (default: 3)

output:
  content: string
  valid: bool
  validation_results: array<{rule, passed, message}>
  attempts: int
  tokens_used: int

token_cost: ~400-1000

workflow:
  1. generate(specification) → initial_content
  2. validate_syntax(initial_content) → syntax_check
  3. validate_data(initial_content, rules) → rule_check
  4. if all_valid:
       return content
     else:
       reason(errors) → improvement_plan
       generate(specification + improvement_plan) → retry
       repeat until valid or max_attempts

example:
  input:
    specification: "TypeScript function to validate email with proper error handling"
    type: "code"
    language: "typescript"
    validation_rules: ["has_type_annotations", "handles_errors", "has_jsdoc"]
  output:
    content: "/**\n * Validates email format\n * @throws {ValidationError}\n */\nfunction validateEmail(email: string): boolean { ... }"
    valid: true
    validation_results:
      - rule: "has_type_annotations"
        passed: true
      - rule: "handles_errors"
        passed: true
      - rule: "has_jsdoc"
        passed: true
    attempts: 1
```

---

### 6. `review_and_suggest`

**Purpose:** Review code/content and provide suggestions.

```yaml
behavior: review_and_suggest
category: quality
composed_of: [read_file, reason, generate, compare]

input:
  file_path: string
  review_criteria: array<string>
  include_fixes: bool (default: true)
  severity_threshold: "all" | "warning" | "critical"

output:
  issues: array<{line, severity, message, suggestion}>
  summary: string
  score: float (0-10)
  fixes: array<{original, suggested}> (if include_fixes)
  tokens_used: int

token_cost: ~400-800

workflow:
  1. read_file(file_path) → content
  2. reason(content, criteria) → issues_analysis
  3. for each issue:
     - if include_fixes:
         generate(fix_suggestion) → suggested_fix
  4. summarize(all_issues) → summary_score

example:
  input:
    file_path: "src/user.service.ts"
    review_criteria: ["security", "performance", "readability"]
    include_fixes: true
  output:
    issues:
      - line: 45
        severity: "critical"
        message: "SQL injection vulnerability"
        suggestion: "Use parameterized queries"
    summary: "1 critical, 2 warnings, 3 style issues found"
    score: 6.5
```

---

### 7. `document_code`

**Purpose:** Generate comprehensive documentation for code.

```yaml
behavior: document_code
category: documentation
composed_of: [read_file, reason, generate, validate_data]

input:
  file_path: string
  doc_type: "jsdoc" | "markdown" | "readme" | "api"
  include_examples: bool (default: true)
  audience: "developer" | "user" | "both"

output:
  documentation: string
  coverage: float (percent of code documented)
  missing_docs: array<string> (undocumented items)
  tokens_used: int

token_cost: ~300-700

workflow:
  1. read_file(file_path) → code_content
  2. reason(code_content) → code_analysis
  3. generate(documentation, doc_type) → initial_docs
  4. if include_examples:
     - generate(examples) → examples
  5. validate_data(docs, completeness_schema) → coverage

example:
  input:
    file_path: "src/auth.service.ts"
    doc_type: "jsdoc"
    include_examples: true
  output:
    documentation: "/**\n * AuthService handles user authentication...\n * @example\n * const auth = new AuthService();\n * await auth.login(credentials);\n */"
    coverage: 0.85
    missing_docs: ["validateToken internal logic"]
```

---

### 8. `refactor_safely`

**Purpose:** Refactor code with full safety checks.

```yaml
behavior: refactor_safely
category: mutation
composed_of: [read_file, search_code, reason, modify_safely, test_change]

input:
  file_path: string
  refactor_type: "extract_function" | "rename" | "move" | "simplify" | "custom"
  target: string (what to refactor)
  desired_outcome: string

output:
  success: bool
  changes_made: array<{file, description}>
  tests_affected: array<string>
  backward_compatible: bool
  rollback_available: bool
  tokens_used: int

token_cost: ~600-1200

workflow:
  1. read_file(file_path) → original_code
  2. search_code(usages of target) → all_usages
  3. reason(refactor plan) → refactor_strategy
  4. for each file affected:
     - modify_safely(file, changes)
  5. test_change(all affected tests)
  6. if tests fail:
     - rollback all changes

example:
  input:
    file_path: "src/user.service.ts"
    refactor_type: "extract_function"
    target: "lines 45-60"
    desired_outcome: "Extract validation logic into validateUserInput function"
  output:
    success: true
    changes_made:
      - file: "src/user.service.ts"
        description: "Extracted validateUserInput function"
    tests_affected: ["user.service.spec.ts"]
    backward_compatible: true
```

---

## Composition Patterns

### Sequential Composition

```yaml
# Each behavior's output feeds the next
sequential:
  - trace_error:
      input: $user_input.error_message
      output_as: $error_analysis
  - generate_with_validation:
      input: 
        specification: "Fix for: $error_analysis.root_cause"
      output_as: $fix
  - test_change:
      input:
        new_content: $fix.content
```

### Parallel Composition

```yaml
# Independent behaviors run simultaneously
parallel:
  investigate:
    behavior: investigate_codebase
    input: { topic: "authentication" }
  review:
    behavior: review_and_suggest
    input: { file_path: "src/auth.ts" }
  
# Results aggregated
aggregate: merge($investigate.output, $review.output)
```

### Conditional Composition

```yaml
# Branch based on intermediate results
conditional:
  check:
    behavior: trace_error
    output_as: $analysis
  
  if: $analysis.confidence > 0.8
  then:
    behavior: refactor_safely
    input: { target: $analysis.root_cause }
  else:
    behavior: ask_user
    input: { question: "Need more details about the error" }
```

---

## Token Budget Reference

| Behavior | Min Tokens | Typical | Max |
|----------|------------|---------|-----|
| trace_error | 400 | 600 | 800 |
| modify_safely | 200 | 300 | 400 |
| test_change | 150 | 200 | 300 |
| investigate_codebase | 500 | 1000 | 1500 |
| generate_with_validation | 400 | 700 | 1000 |
| review_and_suggest | 400 | 600 | 800 |
| document_code | 300 | 500 | 700 |
| refactor_safely | 600 | 900 | 1200 |

---

## Integration with Skills

Skills use compound behaviors as building blocks:

```yaml
# In a skill's SKILL.md
behaviors:
  - trace_error        # Compound
  - modify_safely      # Compound
  - parse_input        # Atomic (when needed individually)

# The skill orchestrates these behaviors
process:
  phase_1:
    behavior: trace_error
    config: { depth: "deep" }
  
  phase_2:
    behavior: modify_safely
    config: { run_tests_after: true }
```

---

*Compound Behaviors Reference v1.0.0 — Workflows from building blocks*
