# Error Handling

This guide covers error handling strategies, retry mechanisms, and failure recovery patterns in Kestra workflows.

## Error Handling Strategies

Kestra provides several mechanisms to handle errors gracefully and ensure workflow resilience.

### Basic Error Tasks

Use the `errors` section to define tasks that run when any task in the workflow fails:

```yaml
id: basic-error-handling
namespace: tutorial

tasks:
  - id: risky-operation
    type: io.kestra.plugin.core.log.Log
    message: "Performing risky operation..."
    
  - id: might-fail
    type: io.kestra.plugin.core.execution.Fail
    message: "This task will always fail"
    
  - id: wont-run
    type: io.kestra.plugin.core.log.Log
    message: "This won't run due to previous failure"

errors:
  - id: handle-failure
    type: io.kestra.plugin.core.log.Log
    message: "❌ Workflow failed! Executing error handling tasks."
    
  - id: cleanup-on-error
    type: io.kestra.plugin.core.log.Log
    message: "🧹 Cleaning up resources after failure"
    
  - id: send-alert
    type: io.kestra.plugin.core.log.Log
    message: "📧 Sending failure notification"
```

### Allow Failure Pattern

Use `allowFailure: true` to continue workflow execution even when specific tasks fail:

```yaml
id: allow-failure-pattern
namespace: tutorial

tasks:
  - id: critical-task
    type: io.kestra.plugin.core.log.Log
    message: "This task must succeed"
    
  - id: optional-notification
    type: io.kestra.plugin.core.execution.Fail
    message: "Notification service is down"
    allowFailure: true  # Don't fail the entire workflow
    
  - id: optional-metrics
    type: io.kestra.plugin.core.log.Log
    message: "Sending metrics to monitoring system"
    allowFailure: true
    
  - id: final-task
    type: io.kestra.plugin.core.log.Log
    message: "✅ Workflow completed despite optional task failures"
```

### Conditional Error Handling

Handle different types of errors with conditional logic:

```yaml
id: conditional-error-handling
namespace: tutorial

inputs:
  - id: environment
    type: SELECT
    values: [development, staging, production]
    defaults: development

tasks:
  - id: data-processing
    type: io.kestra.plugin.core.execution.Fail
    message: "Database connection failed"
    
errors:
  - id: log-error-details
    type: io.kestra.plugin.core.log.Log
    message: |
      Error Details:
      - Environment: {{ inputs.environment }}
      - Failed Task: {{ task.id }}
      - Error Time: {{ now() }}
      - Execution ID: {{ execution.id }}
    
  - id: development-error-handling
    type: io.kestra.plugin.core.log.Log
    message: "🛠️ Development error - detailed logging enabled"
    runIf: "{{ inputs.environment == 'development' }}"
    
  - id: production-error-handling
    type: io.kestra.plugin.core.log.Log
    message: "🚨 Production error - triggering incident response"
    runIf: "{{ inputs.environment == 'production' }}"
    
  - id: staging-error-handling
    type: io.kestra.plugin.core.log.Log
    message: "🔧 Staging error - notifying QA team"
    runIf: "{{ inputs.environment == 'staging' }}"
```

## Retry Mechanisms

### Task-Level Retries

Configure retry behavior for individual tasks:

```yaml
id: task-retry-example
namespace: tutorial

tasks:
  # Constant retry interval
  - id: task-with-constant-retry
    type: io.kestra.plugin.core.execution.Fail
    message: "Task that fails but retries with constant interval"
    retry:
      type: constant
      interval: PT30S        # 30 seconds between retries
      maxAttempt: 3         # Maximum 3 attempts
      
  # Exponential backoff
  - id: task-with-exponential-retry
    type: io.kestra.plugin.core.execution.Fail
    message: "Task with exponential backoff"
    retry:
      type: exponential
      interval: PT10S        # Initial interval: 10 seconds
      maxAttempt: 5         # Maximum 5 attempts
      maxDuration: PT5M     # Stop retrying after 5 minutes
      multiplier: 2.0       # Double the interval each time
      
  # Random retry interval
  - id: task-with-random-retry
    type: io.kestra.plugin.core.execution.Fail
    message: "Task with random retry intervals"
    retry:
      type: random
      interval: PT5S         # Minimum interval
      maxInterval: PT60S     # Maximum interval
      maxAttempt: 4
```

### Advanced Retry Configuration

```yaml
id: advanced-retry-example
namespace: tutorial

variables:
  max_retries: 3
  base_interval: "PT15S"

tasks:
  - id: database-operation
    type: io.kestra.plugin.core.log.Log
    message: "Attempting database operation (attempt {{ taskrun.attemptsCount }})"
    retry:
      type: exponential
      interval: "{{ vars.base_interval }}"
      maxAttempt: "{{ vars.max_retries }}"
      maxDuration: PT10M
      multiplier: 1.5
      
  - id: api-call-with-conditions
    type: io.kestra.plugin.core.log.Log
    message: "Making API call with smart retry logic"
    retry:
      type: constant
      interval: PT20S
      maxAttempt: 5
      # Only retry on specific conditions
      warningOnRetry: true
      
  - id: file-processing
    type: io.kestra.plugin.core.log.Log
    message: "Processing file with retry context"
    retry:
      type: exponential
      interval: PT10S
      maxAttempt: 3
      
  - id: show-retry-context
    type: io.kestra.plugin.core.log.Log
    message: |
      Retry Context Information:
      - Current attempt: {{ taskrun.attemptsCount }}
      - Task run ID: {{ taskrun.id }}
      - Parent task: {{ taskrun.parentId | default('None') }}
```

### Flow-Level Retries

Configure retry behavior for the entire workflow:

```yaml
id: flow-retry-example
namespace: tutorial

# Workflow-level retry configuration
retry:
  behavior: CREATE_NEW_EXECUTION  # or RETRY_FAILED_TASK
  type: exponential
  interval: PT1M
  maxAttempt: 3
  maxDuration: PT30M

inputs:
  - id: operation_mode
    type: SELECT
    values: [normal, aggressive]
    defaults: normal

tasks:
  - id: setup
    type: io.kestra.plugin.core.log.Log
    message: "Setting up workflow (execution attempt {{ execution.originalId != execution.id ? 'retry' : '1' }})"
    
  - id: main-processing
    type: io.kestra.plugin.core.execution.Fail
    message: "Main processing failed - will trigger flow retry"
    runIf: "{{ inputs.operation_mode == 'aggressive' }}"
    
  - id: alternative-processing
    type: io.kestra.plugin.core.log.Log
    message: "Using alternative processing method"
    runIf: "{{ inputs.operation_mode == 'normal' }}"

errors:
  - id: flow-retry-notification
    type: io.kestra.plugin.core.log.Log
    message: |
      Flow retry triggered:
      - Original execution: {{ execution.originalId }}
      - Current execution: {{ execution.id }}
      - Retry attempt in progress
```

## Comprehensive Error Handling Patterns

### Circuit Breaker Pattern

```yaml
id: circuit-breaker-pattern
namespace: tutorial

variables:
  failure_threshold: 3
  circuit_breaker_timeout: "PT5M"

inputs:
  - id: simulate_failures
    type: BOOL
    defaults: false

tasks:
  - id: check-service-health
    type: io.kestra.plugin.core.log.Log
    message: "Checking external service health"
    
  - id: service-call-with-circuit-breaker
    type: io.kestra.plugin.core.flow.If
    condition: "{{ !inputs.simulate_failures }}"
    then:
      - id: successful-service-call
        type: io.kestra.plugin.core.log.Log
        message: "✅ Service call successful"
    else:
      - id: circuit-breaker-open
        type: io.kestra.plugin.core.log.Log
        message: "🔴 Circuit breaker OPEN - service temporarily unavailable"
        
      - id: fallback-service
        type: io.kestra.plugin.core.log.Log
        message: "🔄 Using fallback service"
        
  - id: update-circuit-breaker-state
    type: io.kestra.plugin.core.log.Log
    message: "Circuit breaker state updated based on service response"

errors:
  - id: circuit-breaker-error-handler
    type: io.kestra.plugin.core.log.Log
    message: |
      Circuit breaker error handling:
      - Incrementing failure count
      - Evaluating if circuit should open
      - Current threshold: {{ vars.failure_threshold }}
```

### Graceful Degradation Pattern

```yaml
id: graceful-degradation-pattern
namespace: tutorial

inputs:
  - id: quality_threshold
    type: FLOAT
    defaults: 0.95

tasks:
  - id: primary-data-source
    type: io.kestra.plugin.core.debug.Return
    format: |
      {
        "source": "primary",
        "quality_score": {{ random() }},
        "data_count": 1000
      }
    retry:
      type: constant
      interval: PT10S
      maxAttempt: 2
    allowFailure: true
    
  - id: evaluate-data-quality
    type: io.kestra.plugin.core.flow.If
    condition: "{{ outputs.primary-data-source.value != null and fromJson(outputs.primary-data-source.value).quality_score >= inputs.quality_threshold }}"
    then:
      - id: use-primary-data
        type: io.kestra.plugin.core.log.Log
        message: "✅ Using high-quality primary data (score: {{ fromJson(outputs.primary-data-source.value).quality_score }})"
    else:
      - id: degraded-processing
        type: io.kestra.plugin.core.flow.Parallel
        tasks:
          - id: fallback-data-source
            type: io.kestra.plugin.core.debug.Return
            format: |
              {
                "source": "fallback",
                "quality_score": 0.8,
                "data_count": 500
              }
              
          - id: send-degradation-alert
            type: io.kestra.plugin.core.log.Log
            message: "⚠️ Operating in degraded mode - using fallback data source"
            
      - id: process-with-fallback
        type: io.kestra.plugin.core.log.Log
        message: "Processing with fallback data (reduced quality but available)"

errors:
  - id: complete-failure-handler
    type: io.kestra.plugin.core.log.Log
    message: "❌ Both primary and fallback systems failed - manual intervention required"
```

### Bulk Processing with Error Isolation

```yaml
id: bulk-processing-error-isolation
namespace: tutorial

inputs:
  - id: batch_items
    type: JSON
    defaults: |
      [
        {"id": 1, "data": "valid_item_1"},
        {"id": 2, "data": "invalid_item"},
        {"id": 3, "data": "valid_item_3"},
        {"id": 4, "data": "another_invalid"},
        {"id": 5, "data": "valid_item_5"}
      ]

tasks:
  - id: process-items-with-isolation
    type: io.kestra.plugin.core.flow.EachParallel
    value: "{{ inputs.batch_items }}"
    concurrent: 3
    tasks:
      - id: validate-item
        type: io.kestra.plugin.core.flow.If
        condition: "{{ !taskrun.value.data.contains('invalid') }}"
        then:
          - id: process-valid-item
            type: io.kestra.plugin.core.debug.Return
            format: |
              {
                "item_id": {{ taskrun.value.id }},
                "status": "processed",
                "processed_at": "{{ now() }}",
                "iteration": {{ taskrun.iteration }}
              }
        else:
          - id: handle-invalid-item
            type: io.kestra.plugin.core.debug.Return
            format: |
              {
                "item_id": {{ taskrun.value.id }},
                "status": "skipped",
                "reason": "invalid_data",
                "skipped_at": "{{ now() }}",
                "iteration": {{ taskrun.iteration }}
              }
            allowFailure: true
            
  - id: summarize-results
    type: io.kestra.plugin.core.log.Log
    message: |
      Batch Processing Summary:
      - Total items: {{ inputs.batch_items | length }}
      - Processed items: {{ outputs.process-items-with-isolation.value | selectattr('status', 'eq', 'processed') | list | length }}
      - Skipped items: {{ outputs.process-items-with-isolation.value | selectattr('status', 'eq', 'skipped') | list | length }}
      - Success rate: {{ (outputs.process-items-with-isolation.value | selectattr('status', 'eq', 'processed') | list | length / inputs.batch_items | length * 100) | round(1) }}%

outputs:
  - id: processing_results
    type: JSON
    value: "{{ outputs.process-items-with-isolation.value }}"
    
  - id: success_count
    type: INT
    value: "{{ outputs.process-items-with-isolation.value | selectattr('status', 'eq', 'processed') | list | length }}"
    
  - id: error_count
    type: INT
    value: "{{ outputs.process-items-with-isolation.value | selectattr('status', 'eq', 'skipped') | list | length }}"
```

## Finally Tasks

Use `finally` tasks to ensure cleanup operations run regardless of workflow success or failure:

```yaml
id: finally-tasks-example
namespace: tutorial

inputs:
  - id: simulate_failure
    type: BOOL
    defaults: false

tasks:
  - id: setup-resources
    type: io.kestra.plugin.core.log.Log
    message: "🔧 Setting up temporary resources"
    
  - id: main-processing
    type: io.kestra.plugin.core.flow.If
    condition: "{{ !inputs.simulate_failure }}"
    then:
      - id: successful-processing
        type: io.kestra.plugin.core.log.Log
        message: "✅ Processing completed successfully"
    else:
      - id: simulated-failure
        type: io.kestra.plugin.core.execution.Fail
        message: "💥 Simulated failure occurred"

finally:
  - id: cleanup-resources
    type: io.kestra.plugin.core.log.Log
    message: "🧹 Cleaning up resources (runs regardless of success/failure)"
    
  - id: save-execution-log
    type: io.kestra.plugin.core.log.Log
    message: |
      Execution Summary:
      - Execution ID: {{ execution.id }}
      - Final State: {{ execution.state }}
      - End Time: {{ now() }}
      
  - id: conditional-cleanup
    type: io.kestra.plugin.core.log.Log
    message: "🗑️ Performing additional cleanup for failed executions"
    runIf: "{{ execution.state.name() == 'FAILED' }}"

errors:
  - id: error-logging
    type: io.kestra.plugin.core.log.Log
    message: "❌ Error occurred in main workflow"
```

## Error Context and Debugging

### Comprehensive Error Information

```yaml
id: error-context-example
namespace: tutorial

tasks:
  - id: operation-that-fails
    type: io.kestra.plugin.core.execution.Fail
    message: "This operation will fail for demonstration"

errors:
  - id: detailed-error-context
    type: io.kestra.plugin.core.log.Log
    message: |
      🔍 Detailed Error Context:
      
      Execution Information:
      - Execution ID: {{ execution.id }}
      - Flow ID: {{ flow.id }}
      - Namespace: {{ flow.namespace }}
      - Started At: {{ execution.startDate }}
      - Current State: {{ execution.state }}
      
      Task Information:
      - Failed Task ID: {{ task.id }}
      - Task Type: {{ task.type }}
      
      Environment:
      - Execution Context: {{ printContext() | truncate(500) }}
      
      Error Logs:
      {{ errorLogs() | truncate(1000) }}
      
  - id: create-error-report
    type: io.kestra.plugin.core.debug.Return
    format: |
      {
        "error_report": {
          "execution_id": "{{ execution.id }}",
          "failed_task": "{{ task.id }}",
          "timestamp": "{{ now() }}",
          "flow_info": {
            "id": "{{ flow.id }}",
            "namespace": "{{ flow.namespace }}",
            "revision": {{ flow.revision }}
          }
        }
      }
```

## Best Practices

### 1. Use Appropriate Error Handling Levels

```yaml
# Task-level for specific operations
- id: api-call
  type: io.kestra.plugin.core.log.Log
  message: "Calling external API"
  retry:
    type: exponential
    maxAttempt: 3
  allowFailure: true  # Don't fail workflow for non-critical APIs

# Flow-level for workflow resilience
retry:
  behavior: CREATE_NEW_EXECUTION
  maxAttempt: 2

# Global error handling
errors:
  - id: cleanup-and-alert
```

### 2. Implement Progressive Error Handling

```yaml
tasks:
  - id: quick-retry-operation
    retry:
      type: constant
      interval: PT5S
      maxAttempt: 3
    allowFailure: true
    
  - id: fallback-if-needed
    type: io.kestra.plugin.core.log.Log
    message: "Using fallback approach"
    runIf: "{{ outputs.quick-retry-operation.value == null }}"
```

### 3. Provide Meaningful Error Messages

```yaml
errors:
  - id: user-friendly-error
    type: io.kestra.plugin.core.log.Log
    message: |
      ❌ Workflow Failed: Data Processing Pipeline
      
      What happened: The main data processing task failed after {{ taskrun.attemptsCount }} attempts
      
      Impact: New data will not be available until this is resolved
      
      Next steps:
      1. Check data source connectivity
      2. Validate input data format
      3. Review system resource usage
      
      Technical details:
      - Execution ID: {{ execution.id }}
      - Failed at: {{ now() }}
      - Error context: {{ errorLogs() | truncate(200) }}
```

### 4. Use Circuit Breakers for External Dependencies

```yaml
variables:
  service_health_check_url: "https://api.example.com/health"
  max_consecutive_failures: 5

tasks:
  - id: health-check
    type: io.kestra.plugin.core.log.Log
    message: "Checking service health before processing"
    
  - id: conditional-processing
    type: io.kestra.plugin.core.flow.If
    condition: "{{ vars.service_healthy | default(true) }}"
    then:
      - id: normal-processing
    else:
      - id: circuit-breaker-fallback
```

## Next Steps

- Learn about [Triggers and Scheduling](triggers-scheduling.md) for automated execution
- Explore [Best Practices](best-practices.md) for maintainable workflows
- Review [Flow Control](flow-control.md) for advanced execution patterns