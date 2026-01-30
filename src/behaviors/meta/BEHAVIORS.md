# Meta Behaviors Reference

> **Component:** Behavior Layer  
> **Type:** Meta (Controls behavior execution flow)  
> **Version:** 1.0.0

---

## Overview

Meta behaviors are special behaviors that control HOW other behaviors execute. They manage flow, orchestration, and execution patterns rather than performing actions directly.

**Properties of Meta Behaviors:**
- Control execution flow
- Manage behavior composition
- Handle error recovery
- Enable dynamic behavior selection
- Provide execution monitoring

---

## Catalog of Meta Behaviors

### 1. `chain`

**Purpose:** Execute behaviors in strict sequence, passing output to input.

```yaml
behavior: chain
category: flow_control

input:
  behaviors: array<{behavior, config}>
  pass_through: "output" | "merged" | "last_only"
  stop_on_error: bool (default: true)

output:
  results: array<{behavior, output, success}>
  final_output: any
  chain_success: bool
  total_tokens: int

mechanism:
  for each behavior in sequence:
    output[n] = execute(behavior[n], input=output[n-1])
    if error and stop_on_error:
      break

example:
  input:
    behaviors:
      - behavior: parse_input
        config: { parse_type: "error" }
      - behavior: search_code
        config: { scope: "project" }
      - behavior: reason
        config: { depth: "medium" }
    pass_through: "merged"
  
  execution:
    parse_input → { keywords: ["login", "null"] }
        ↓
    search_code → { results: [file1, file2] }
        ↓
    reason → { conclusion: "Null check missing" }
```

---

### 2. `parallel`

**Purpose:** Execute independent behaviors simultaneously.

```yaml
behavior: parallel
category: flow_control

input:
  behaviors: array<{name, behavior, config}>
  wait_strategy: "all" | "first" | "majority"
  timeout_ms: int (default: 30000)
  fail_fast: bool (default: false)

output:
  results: object<name, output>
  completed: array<string>
  failed: array<{name, error}>
  total_tokens: int

mechanism:
  spawn all behaviors concurrently
  wait according to strategy
  aggregate results

example:
  input:
    behaviors:
      - name: files_analysis
        behavior: read_file
        config: { path: "src/auth.ts" }
      - name: tests_analysis
        behavior: read_file
        config: { path: "tests/auth.spec.ts" }
      - name: deps_analysis
        behavior: search_code
        config: { query: "import.*auth" }
    wait_strategy: "all"
  
  output:
    results:
      files_analysis: { content: "..." }
      tests_analysis: { content: "..." }
      deps_analysis: { results: [...] }
```

---

### 3. `conditional`

**Purpose:** Branch execution based on conditions.

```yaml
behavior: conditional
category: flow_control

input:
  condition: string | object  # Expression or behavior
  evaluate_with: "expression" | "behavior"
  if_true: {behavior, config}
  if_false: {behavior, config}
  default: {behavior, config} (optional)

output:
  branch_taken: "if_true" | "if_false" | "default"
  condition_result: any
  branch_output: any
  total_tokens: int

mechanism:
  evaluate condition
  select branch
  execute selected behavior

example:
  input:
    condition:
      behavior: validate_syntax
      config: { code: $previous_output.generated_code }
    evaluate_with: "behavior"
    if_true:
      behavior: write_file
      config: { content: $previous_output.generated_code }
    if_false:
      behavior: generate
      config: { specification: "Fix syntax errors in: ..." }
```

---

### 4. `loop`

**Purpose:** Repeat behavior until condition is met.

```yaml
behavior: loop
category: flow_control

input:
  behavior: {behavior, config}
  until: string | object  # Stop condition
  max_iterations: int (default: 10)
  collect_results: bool (default: true)
  pass_previous: bool (default: true)

output:
  iterations: int
  results: array<any> (if collect_results)
  final_output: any
  stopped_by: "condition" | "max_iterations" | "error"
  total_tokens: int

mechanism:
  repeat:
    execute behavior
    check condition
    if met or max reached: stop

example:
  input:
    behavior:
      behavior: generate_with_validation
      config: { type: "code" }
    until:
      expression: "$output.valid == true"
    max_iterations: 3
  
  output:
    iterations: 2
    results: [{ valid: false, ... }, { valid: true, ... }]
    final_output: { valid: true, content: "..." }
    stopped_by: "condition"
```

---

### 5. `fallback`

**Purpose:** Try behaviors in order until one succeeds.

```yaml
behavior: fallback
category: error_handling

input:
  behaviors: array<{behavior, config}>
  success_condition: string  # What counts as success
  return_first_success: bool (default: true)

output:
  success: bool
  successful_behavior: string (if success)
  attempts: array<{behavior, result, success}>
  final_output: any
  total_tokens: int

mechanism:
  for each behavior:
    try execute
    if success_condition met:
      return result
  return failure

example:
  input:
    behaviors:
      - behavior: search_code
        config: { type: "semantic" }  # Try semantic first
      - behavior: search_code
        config: { type: "regex" }     # Fall back to regex
      - behavior: search_code
        config: { type: "text" }      # Last resort: text
    success_condition: "$output.results.length > 0"
```

---

### 6. `retry`

**Purpose:** Retry a behavior with configurable backoff.

```yaml
behavior: retry
category: error_handling

input:
  behavior: {behavior, config}
  max_retries: int (default: 3)
  backoff: "none" | "linear" | "exponential"
  backoff_base_ms: int (default: 1000)
  retry_on: array<string>  # Error types to retry

output:
  success: bool
  attempts: int
  final_output: any
  error_history: array<{attempt, error}>
  total_tokens: int

mechanism:
  attempt = 0
  while attempt < max_retries:
    try execute
    if success: return
    wait(backoff)
    attempt++

example:
  input:
    behavior:
      behavior: run_command
      config: { command: "npm test" }
    max_retries: 3
    backoff: "exponential"
    retry_on: ["TIMEOUT", "NETWORK_ERROR"]
```

---

### 7. `guard`

**Purpose:** Execute behavior only if preconditions are met.

```yaml
behavior: guard
category: flow_control

input:
  preconditions: array<{check, config}>
  guarded_behavior: {behavior, config}
  on_fail: "skip" | "error" | "alternative"
  alternative: {behavior, config} (if on_fail: "alternative")

output:
  preconditions_met: bool
  precondition_results: array<{check, passed, reason}>
  behavior_executed: bool
  behavior_output: any (if executed)
  total_tokens: int

mechanism:
  check all preconditions
  if all pass:
    execute guarded_behavior
  else:
    handle according to on_fail

example:
  input:
    preconditions:
      - check: file_exists
        config: { path: "src/auth.ts" }
      - check: has_permission
        config: { action: "write" }
    guarded_behavior:
      behavior: modify_safely
      config: { file_path: "src/auth.ts" }
    on_fail: "error"
```

---

### 8. `pipeline`

**Purpose:** Build complex multi-stage pipelines with named stages.

```yaml
behavior: pipeline
category: orchestration

input:
  stages: array<{name, behavior, config, depends_on}>
  parallel_stages: bool (default: true)
  checkpoint: bool (default: false)

output:
  stage_results: object<stage_name, output>
  pipeline_success: bool
  failed_stage: string (if failed)
  execution_order: array<string>
  total_tokens: int

mechanism:
  build dependency graph
  execute stages respecting dependencies
  run independent stages in parallel if enabled

example:
  input:
    stages:
      - name: analyze
        behavior: investigate_codebase
        config: { topic: "authentication" }
        depends_on: []
      - name: review
        behavior: review_and_suggest
        config: { file_path: "src/auth.ts" }
        depends_on: []
      - name: plan
        behavior: reason
        config: { premise: "$analyze + $review" }
        depends_on: [analyze, review]
      - name: implement
        behavior: refactor_safely
        config: { based_on: "$plan" }
        depends_on: [plan]
    parallel_stages: true
```

---

### 9. `map`

**Purpose:** Apply a behavior to each item in a collection.

```yaml
behavior: map
category: iteration

input:
  collection: array<any>
  behavior: {behavior, config_template}
  parallel: bool (default: true)
  max_concurrent: int (default: 5)
  continue_on_error: bool (default: false)

output:
  results: array<{item, output, success}>
  successful: int
  failed: int
  total_tokens: int

mechanism:
  for each item in collection:
    config = template.interpolate(item)
    execute behavior with config

example:
  input:
    collection: ["src/auth.ts", "src/user.ts", "src/api.ts"]
    behavior:
      behavior: review_and_suggest
      config_template:
        file_path: "$item"
        review_criteria: ["security"]
    parallel: true
    max_concurrent: 3
```

---

### 10. `reduce`

**Purpose:** Aggregate results from multiple behaviors into single output.

```yaml
behavior: reduce
category: aggregation

input:
  behaviors: array<{behavior, config}>
  reducer: string  # Reduction function
  initial_value: any

output:
  final_value: any
  intermediate_values: array<any>
  total_tokens: int

mechanism:
  accumulator = initial_value
  for each behavior:
    result = execute(behavior)
    accumulator = reducer(accumulator, result)
  return accumulator

example:
  input:
    behaviors:
      - behavior: read_file
        config: { path: "src/auth.ts" }
      - behavior: read_file
        config: { path: "src/user.ts" }
    reducer: "merge_summaries"
    initial_value: { total_lines: 0, total_functions: 0 }
  
  output:
    final_value:
      total_lines: 450
      total_functions: 23
```

---

## Composition Examples

### Complex Debugging Pipeline

```yaml
name: comprehensive_debug
type: pipeline

stages:
  - name: gather_context
    meta: parallel
    behaviors:
      - behavior: trace_error
      - behavior: investigate_codebase
      - behavior: search_code
  
  - name: analyze
    meta: chain
    depends_on: [gather_context]
    behaviors:
      - behavior: reason
      - behavior: generate_with_validation
  
  - name: implement
    meta: guard
    depends_on: [analyze]
    preconditions:
      - { check: confidence, threshold: 0.8 }
    guarded: modify_safely
  
  - name: verify
    meta: retry
    depends_on: [implement]
    behavior: test_change
    max_retries: 2
```

### Batch Code Review

```yaml
name: batch_review
type: map

input:
  files: ["auth.ts", "user.ts", "api.ts"]

process:
  meta: map
  collection: $files
  behavior:
    meta: chain
    behaviors:
      - behavior: review_and_suggest
      - behavior: document_code
  
  post_process:
    meta: reduce
    reducer: aggregate_findings
```

---

## Token Budget Impact

| Meta Behavior | Overhead | Notes |
|---------------|----------|-------|
| chain | ~10 tokens/step | Minimal overhead |
| parallel | ~20 tokens total | Fixed overhead |
| conditional | ~30 tokens | Condition eval |
| loop | ~15 tokens/iter | Condition check each |
| fallback | ~10 tokens/attempt | Try-catch overhead |
| retry | ~10 tokens/retry | Similar to fallback |
| guard | ~50 tokens | Precondition checks |
| pipeline | ~100 tokens | Graph management |
| map | ~20 + n×5 tokens | Per-item overhead |
| reduce | ~30 tokens | Accumulator management |

---

*Meta Behaviors Reference v1.0.0 — Orchestrating intelligent workflows*
