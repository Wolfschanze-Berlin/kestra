# Triggers and Scheduling

This guide covers how to automate workflow execution using triggers, including scheduling, event-based triggers, and flow dependencies.

## Schedule Triggers

Schedule triggers automatically execute workflows based on time-based conditions using cron expressions.

### Basic Scheduling

```yaml
id: basic-schedule-example
namespace: tutorial

triggers:
  - id: daily-report
    type: io.kestra.plugin.core.trigger.Schedule
    cron: "0 9 * * *"  # Every day at 9:00 AM
    
tasks:
  - id: generate-daily-report
    type: io.kestra.plugin.core.log.Log
    message: |
      📊 Generating daily report
      - Triggered at: {{ trigger.date }}
      - Next run: {{ trigger.next }}
      - Previous run: {{ trigger.previous | default('First run') }}
```

### Advanced Scheduling

```yaml
id: advanced-schedule-example
namespace: tutorial

triggers:
  # Multiple triggers for different schedules
  - id: weekday-morning
    type: io.kestra.plugin.core.trigger.Schedule
    cron: "0 8 * * 1-5"  # Monday to Friday at 8:00 AM
    timezone: "America/New_York"
    
  - id: weekend-maintenance
    type: io.kestra.plugin.core.trigger.Schedule
    cron: "0 2 * * 6"    # Every Saturday at 2:00 AM
    timezone: "UTC"
    
  - id: month-end-report
    type: io.kestra.plugin.core.trigger.Schedule
    cron: "0 23 L * *"   # Last day of every month at 11:00 PM

tasks:
  - id: determine-schedule-type
    type: io.kestra.plugin.core.log.Log
    message: |
      Schedule Information:
      - Trigger ID: {{ trigger.id }}
      - Execution time: {{ trigger.date }}
      - Day of week: {{ trigger.date | date('EEEE') }}
      - Is weekend: {{ trigger.date | date('u') >= 6 }}
      
  - id: weekday-processing
    type: io.kestra.plugin.core.log.Log
    message: "🏢 Running weekday business processing"
    runIf: "{{ trigger.date | date('u') <= 5 }}"
    
  - id: weekend-maintenance
    type: io.kestra.plugin.core.log.Log
    message: "🔧 Running weekend maintenance tasks"
    runIf: "{{ trigger.date | date('u') >= 6 }}"
```

### Cron Expression Examples

```yaml
id: cron-examples
namespace: tutorial
description: "Common cron expression patterns"

triggers:
  # Every minute
  - id: every-minute
    type: io.kestra.plugin.core.trigger.Schedule
    cron: "* * * * *"
    disabled: true  # Disabled for demonstration
    
  # Every 15 minutes
  - id: every-15-minutes
    type: io.kestra.plugin.core.trigger.Schedule
    cron: "*/15 * * * *"
    disabled: true
    
  # Every hour at minute 30
  - id: every-hour-at-30
    type: io.kestra.plugin.core.trigger.Schedule
    cron: "30 * * * *"
    disabled: true
    
  # Every day at 2:30 AM
  - id: daily-at-230am
    type: io.kestra.plugin.core.trigger.Schedule
    cron: "30 2 * * *"
    disabled: true
    
  # Every Monday at 9:00 AM
  - id: every-monday
    type: io.kestra.plugin.core.trigger.Schedule
    cron: "0 9 * * 1"
    disabled: true
    
  # First day of every month at midnight
  - id: monthly-first-day
    type: io.kestra.plugin.core.trigger.Schedule
    cron: "0 0 1 * *"
    disabled: true
    
  # Every quarter (Jan, Apr, Jul, Oct) on the 1st at 6 AM
  - id: quarterly
    type: io.kestra.plugin.core.trigger.Schedule
    cron: "0 6 1 */3 *"
    disabled: true
    
  # Business hours: Monday-Friday, 9 AM to 5 PM, every hour
  - id: business-hours
    type: io.kestra.plugin.core.trigger.Schedule
    cron: "0 9-17 * * 1-5"

tasks:
  - id: show-trigger-info
    type: io.kestra.plugin.core.log.Log
    message: |
      Cron Trigger Information:
      - Pattern: Business hours (9 AM - 5 PM, Mon-Fri)
      - Current trigger: {{ trigger.date }}
      - Is business hour: {{ trigger.date | date('H') >= 9 and trigger.date | date('H') <= 17 and trigger.date | date('u') <= 5 }}
```

### Schedule with Backfill

```yaml
id: schedule-with-backfill
namespace: tutorial

triggers:
  - id: daily-with-backfill
    type: io.kestra.plugin.core.trigger.Schedule
    cron: "0 6 * * *"  # Every day at 6:00 AM
    backfill:
      start: "2024-01-01T00:00:00Z"  # Start backfilling from this date
      
tasks:
  - id: process-daily-data
    type: io.kestra.plugin.core.log.Log
    message: |
      Processing data for date: {{ trigger.date | date('yyyy-MM-dd') }}
      - Is backfill: {{ trigger.date < now() }}
      - Execution context: {{ execution.id }}
      
  - id: handle-backfill
    type: io.kestra.plugin.core.flow.If
    condition: "{{ trigger.date < now() | dateAdd(-1, 'DAYS') }}"
    then:
      - id: backfill-processing
        type: io.kestra.plugin.core.log.Log
        message: "🔄 Running backfill for {{ trigger.date | date('yyyy-MM-dd') }}"
    else:
      - id: current-processing
        type: io.kestra.plugin.core.log.Log
        message: "⚡ Running current processing for {{ trigger.date | date('yyyy-MM-dd') }}"
```

## Flow Triggers

Flow triggers execute a workflow when another workflow completes, creating workflow dependencies.

### Basic Flow Trigger

```yaml
id: flow-trigger-listener
namespace: tutorial

triggers:
  - id: wait-for-data-pipeline
    type: io.kestra.plugin.core.trigger.Flow
    conditions:
      - type: io.kestra.plugin.core.condition.ExecutionFlowCondition
        namespace: tutorial
        flowId: data-pipeline-source
        
  - id: wait-for-multiple-flows
    type: io.kestra.plugin.core.trigger.Flow
    conditions:
      - type: io.kestra.plugin.core.condition.ExecutionFlowCondition
        namespace: tutorial
        flowId: etl-pipeline-a
      - type: io.kestra.plugin.core.condition.ExecutionFlowCondition
        namespace: tutorial
        flowId: etl-pipeline-b

tasks:
  - id: process-after-dependencies
    type: io.kestra.plugin.core.log.Log
    message: |
      Dependent Flow Execution:
      - Triggered by: {{ trigger.executionId }}
      - Source namespace: {{ trigger.namespace }}
      - Source flow: {{ trigger.flowId }}
      - Source flow revision: {{ trigger.flowRevision }}
      - Trigger timestamp: {{ trigger.date }}
      
  - id: aggregate-results
    type: io.kestra.plugin.core.log.Log
    message: "📊 Aggregating results from upstream workflows"
```

### Conditional Flow Triggers

```yaml
id: conditional-flow-trigger
namespace: tutorial

triggers:
  - id: success-only-trigger
    type: io.kestra.plugin.core.trigger.Flow
    conditions:
      - type: io.kestra.plugin.core.condition.ExecutionFlowCondition
        namespace: tutorial
        flowId: upstream-flow
      - type: io.kestra.plugin.core.condition.ExecutionStatusCondition
        in: [SUCCESS]  # Only trigger on successful executions
        
  - id: failure-handling-trigger
    type: io.kestra.plugin.core.trigger.Flow
    conditions:
      - type: io.kestra.plugin.core.condition.ExecutionFlowCondition
        namespace: tutorial
        flowId: critical-pipeline
      - type: io.kestra.plugin.core.condition.ExecutionStatusCondition
        in: [FAILED, WARNING]  # Trigger on failures or warnings

tasks:
  - id: handle-upstream-result
    type: io.kestra.plugin.core.flow.Switch
    value: "{{ trigger.executionState }}"
    cases:
      SUCCESS:
        - id: handle-success
          type: io.kestra.plugin.core.log.Log
          message: "✅ Upstream flow succeeded - proceeding with normal processing"
          
      FAILED:
        - id: handle-failure
          type: io.kestra.plugin.core.log.Log
          message: "❌ Upstream flow failed - initiating recovery procedures"
          
      WARNING:
        - id: handle-warning
          type: io.kestra.plugin.core.log.Log
          message: "⚠️ Upstream flow completed with warnings - proceeding with caution"
```

### Flow Chain Example

Here are three workflows that demonstrate flow chaining:

#### Source Data Pipeline
```yaml
id: data-pipeline-source
namespace: tutorial

tasks:
  - id: extract-data
    type: io.kestra.plugin.core.debug.Return
    format: |
      {
        "extracted_records": 10000,
        "extraction_time": "{{ now() }}",
        "data_quality": "high"
      }
      
  - id: validate-data
    type: io.kestra.plugin.core.log.Log
    message: "✅ Data validation completed - {{ fromJson(outputs.extract-data.value).extracted_records }} records"

outputs:
  - id: extraction_summary
    type: JSON
    value: "{{ outputs.extract-data.value }}"
```

#### Transformation Pipeline (triggered by source)
```yaml
id: transformation-pipeline
namespace: tutorial

triggers:
  - id: wait-for-source-data
    type: io.kestra.plugin.core.trigger.Flow
    conditions:
      - type: io.kestra.plugin.core.condition.ExecutionFlowCondition
        namespace: tutorial
        flowId: data-pipeline-source

tasks:
  - id: transform-data
    type: io.kestra.plugin.core.debug.Return
    format: |
      {
        "input_records": {{ random() * 10000 | round }},
        "transformed_records": {{ random() * 9500 | round }},
        "transformation_time": "{{ now() }}",
        "upstream_execution": "{{ trigger.executionId }}"
      }

outputs:
  - id: transformation_summary
    type: JSON
    value: "{{ outputs.transform-data.value }}"
```

#### Reporting Pipeline (triggered by transformation)
```yaml
id: reporting-pipeline
namespace: tutorial

triggers:
  - id: wait-for-transformation
    type: io.kestra.plugin.core.trigger.Flow
    conditions:
      - type: io.kestra.plugin.core.condition.ExecutionFlowCondition
        namespace: tutorial
        flowId: transformation-pipeline

tasks:
  - id: generate-report
    type: io.kestra.plugin.core.log.Log
    message: |
      📊 Final Report Generated
      - Upstream execution: {{ trigger.executionId }}
      - Report timestamp: {{ now() }}
      - Pipeline chain completed successfully
```

## Webhook Triggers

Webhook triggers allow external systems to trigger workflows via HTTP requests.

### Basic Webhook

```yaml
id: webhook-trigger-example
namespace: tutorial

triggers:
  - id: api-webhook
    type: io.kestra.plugin.core.trigger.Webhook
    key: "my-webhook-key"  # Optional authentication key

tasks:
  - id: process-webhook-data
    type: io.kestra.plugin.core.log.Log
    message: |
      Webhook received:
      - Timestamp: {{ trigger.date }}
      - Headers: {{ trigger.headers | toJson }}
      - Body: {{ trigger.body | default('No body') }}
      - Query parameters: {{ trigger.parameters | default({}) | toJson }}
      
  - id: validate-webhook-payload
    type: io.kestra.plugin.core.flow.If
    condition: "{{ trigger.body != null }}"
    then:
      - id: process-payload
        type: io.kestra.plugin.core.debug.Return
        format: |
          {
            "processed_at": "{{ now() }}",
            "webhook_data": {{ trigger.body | toJson }},
            "processing_status": "completed"
          }
    else:
      - id: handle-empty-payload
        type: io.kestra.plugin.core.log.Log
        message: "⚠️ Webhook received with empty payload"
```

### Webhook with Authentication

```yaml
id: secure-webhook-example
namespace: tutorial

triggers:
  - id: secure-webhook
    type: io.kestra.plugin.core.trigger.Webhook
    key: "{{ secret('WEBHOOK_SECRET_KEY') }}"
    
tasks:
  - id: authenticated-processing
    type: io.kestra.plugin.core.log.Log
    message: |
      Secure webhook processing:
      - Authentication verified
      - Source IP: {{ trigger.headers['X-Forwarded-For'] | default('Unknown') }}
      - User-Agent: {{ trigger.headers['User-Agent'] | default('Unknown') }}
      - Content-Type: {{ trigger.headers['Content-Type'] | default('Unknown') }}
      
  - id: parse-webhook-content
    type: io.kestra.plugin.core.flow.Switch
    value: "{{ trigger.headers['Content-Type'] | default('') | split(';') | first }}"
    cases:
      "application/json":
        - id: process-json
          type: io.kestra.plugin.core.log.Log
          message: "Processing JSON payload: {{ trigger.body | fromJson | toJson }}"
          
      "application/x-www-form-urlencoded":
        - id: process-form-data
          type: io.kestra.plugin.core.log.Log
          message: "Processing form data: {{ trigger.parameters | toJson }}"
          
    defaults:
      - id: process-raw-data
        type: io.kestra.plugin.core.log.Log
        message: "Processing raw data: {{ trigger.body }}"
```

## Composite Trigger Patterns

### Time and Condition Based Triggers

```yaml
id: composite-trigger-example
namespace: tutorial

triggers:
  # Trigger during business hours OR when explicitly requested
  - id: business-hours-or-manual
    type: io.kestra.plugin.core.trigger.Schedule
    cron: "0 9-17 * * 1-5"  # Business hours
    
  - id: manual-override
    type: io.kestra.plugin.core.trigger.Webhook
    key: "manual-trigger-key"

tasks:
  - id: determine-trigger-source
    type: io.kestra.plugin.core.flow.Switch
    value: "{{ trigger.type }}"
    cases:
      "io.kestra.plugin.core.trigger.Schedule":
        - id: scheduled-processing
          type: io.kestra.plugin.core.log.Log
          message: "📅 Scheduled execution during business hours: {{ trigger.date }}"
          
      "io.kestra.plugin.core.trigger.Webhook":
        - id: manual-processing
          type: io.kestra.plugin.core.log.Log
          message: "👤 Manual trigger received: {{ trigger.body | default('No additional data') }}"
          
  - id: common-processing
    type: io.kestra.plugin.core.log.Log
    message: "🔄 Executing common processing logic regardless of trigger source"
```

## Advanced Scheduling Patterns

### Dynamic Scheduling

```yaml
id: dynamic-schedule-example
namespace: tutorial

variables:
  # Schedule varies by environment
  schedule_cron: "{{ vars.environment == 'prod' ? '0 2 * * *' : '0 */6 * * *' }}"
  environment: "prod"

triggers:
  - id: environment-based-schedule
    type: io.kestra.plugin.core.trigger.Schedule
    cron: "0 2 * * *"  # Production: daily at 2 AM
    disabled: "{{ vars.environment != 'prod' }}"
    
  - id: development-schedule
    type: io.kestra.plugin.core.trigger.Schedule
    cron: "0 */6 * * *"  # Development: every 6 hours
    disabled: "{{ vars.environment == 'prod' }}"

tasks:
  - id: environment-aware-processing
    type: io.kestra.plugin.core.log.Log
    message: |
      Environment-specific execution:
      - Environment: {{ vars.environment }}
      - Schedule: {{ vars.environment == 'prod' ? 'Daily at 2 AM' : 'Every 6 hours' }}
      - Trigger time: {{ trigger.date }}
```

### Multi-Region Scheduling

```yaml
id: multi-region-schedule
namespace: tutorial

triggers:
  # US East Coast processing
  - id: us-east-schedule
    type: io.kestra.plugin.core.trigger.Schedule
    cron: "0 6 * * *"  # 6 AM Eastern
    timezone: "America/New_York"
    
  # European processing
  - id: eu-schedule
    type: io.kestra.plugin.core.trigger.Schedule
    cron: "0 9 * * *"  # 9 AM Central European Time
    timezone: "Europe/Paris"
    
  # Asia Pacific processing
  - id: apac-schedule
    type: io.kestra.plugin.core.trigger.Schedule
    cron: "0 10 * * *"  # 10 AM JST
    timezone: "Asia/Tokyo"

tasks:
  - id: region-specific-processing
    type: io.kestra.plugin.core.log.Log
    message: |
      Regional Processing:
      - Local trigger time: {{ trigger.date }}
      - UTC time: {{ trigger.date | date('yyyy-MM-dd HH:mm:ss', 'UTC') }}
      - Timezone: {{ trigger.date | date('z') }}
      
  - id: determine-region
    type: io.kestra.plugin.core.flow.Switch
    value: "{{ trigger.date | date('z') }}"
    cases:
      "EST":
        - id: us-processing
          type: io.kestra.plugin.core.log.Log
          message: "🇺🇸 Processing US East Coast data"
      "CET":
        - id: eu-processing
          type: io.kestra.plugin.core.log.Log
          message: "🇪🇺 Processing European data"
      "JST":
        - id: apac-processing
          type: io.kestra.plugin.core.log.Log
          message: "🌏 Processing Asia Pacific data"
```

## Monitoring and Alerting Patterns

### Trigger Health Monitoring

```yaml
id: trigger-health-monitoring
namespace: tutorial

triggers:
  - id: health-check-schedule
    type: io.kestra.plugin.core.trigger.Schedule
    cron: "*/5 * * * *"  # Every 5 minutes

tasks:
  - id: check-trigger-health
    type: io.kestra.plugin.core.log.Log
    message: |
      Trigger Health Check:
      - Current time: {{ now() }}
      - Expected trigger: {{ trigger.date }}
      - Delay: {{ (now() | timestamp) - (trigger.date | timestamp) }} seconds
      - Is on time: {{ ((now() | timestamp) - (trigger.date | timestamp)) < 60 }}
      
  - id: alert-on-delay
    type: io.kestra.plugin.core.log.Log
    message: "⚠️ Trigger delay detected - investigation required"
    runIf: "{{ ((now() | timestamp) - (trigger.date | timestamp)) > 300 }}"  # 5 minute delay
```

### Missed Execution Detection

```yaml
id: missed-execution-detection
namespace: tutorial

triggers:
  - id: daily-critical-job
    type: io.kestra.plugin.core.trigger.Schedule
    cron: "0 3 * * *"  # Daily at 3 AM

variables:
  max_execution_delay_hours: 2

tasks:
  - id: check-execution-timing
    type: io.kestra.plugin.core.log.Log
    message: |
      Execution Timing Check:
      - Scheduled for: {{ trigger.date }}
      - Actually started: {{ execution.startDate }}
      - Delay: {{ (execution.startDate | timestamp) - (trigger.date | timestamp) }} seconds
      
  - id: validate-execution-window
    type: io.kestra.plugin.core.flow.If
    condition: "{{ ((execution.startDate | timestamp) - (trigger.date | timestamp)) > (vars.max_execution_delay_hours * 3600) }}"
    then:
      - id: alert-missed-window
        type: io.kestra.plugin.core.log.Log
        message: |
          🚨 CRITICAL: Execution outside acceptable window
          - Max allowed delay: {{ vars.max_execution_delay_hours }} hours
          - Actual delay: {{ ((execution.startDate | timestamp) - (trigger.date | timestamp)) / 3600 | round(2) }} hours
    else:
      - id: normal-execution
        type: io.kestra.plugin.core.log.Log
        message: "✅ Execution within acceptable timing window"
```

## Best Practices

### 1. Use Appropriate Trigger Types

```yaml
# Use Schedule for time-based automation
triggers:
  - id: regular-maintenance
    type: io.kestra.plugin.core.trigger.Schedule
    cron: "0 2 * * 0"  # Weekly on Sunday at 2 AM

# Use Flow triggers for workflow dependencies
triggers:
  - id: wait-for-upstream
    type: io.kestra.plugin.core.trigger.Flow
    conditions:
      - type: io.kestra.plugin.core.condition.ExecutionFlowCondition
        flowId: data-preparation

# Use Webhook for external integrations
triggers:
  - id: external-api-integration
    type: io.kestra.plugin.core.trigger.Webhook
    key: "{{ secret('API_WEBHOOK_KEY') }}"
```

### 2. Handle Timezone Considerations

```yaml
triggers:
  - id: timezone-aware-schedule
    type: io.kestra.plugin.core.trigger.Schedule
    cron: "0 9 * * 1-5"  # 9 AM business days
    timezone: "America/New_York"  # Explicitly set timezone

tasks:
  - id: log-timezone-info
    type: io.kestra.plugin.core.log.Log
    message: |
      Timezone Information:
      - Local time: {{ trigger.date }}
      - UTC time: {{ trigger.date | date('yyyy-MM-dd HH:mm:ss', 'UTC') }}
      - Timezone: {{ trigger.date | date('z') }}
```

### 3. Implement Trigger Monitoring

```yaml
tasks:
  - id: log-trigger-context
    type: io.kestra.plugin.core.log.Log
    message: |
      Trigger Context:
      - Type: {{ trigger.type }}
      - Scheduled: {{ trigger.date }}
      - Actual start: {{ execution.startDate }}
      - Previous run: {{ trigger.previous | default('First run') }}
      - Next run: {{ trigger.next | default('Not scheduled') }}
```

### 4. Use Descriptive Trigger IDs

```yaml
triggers:
  # Good - descriptive and meaningful
  - id: daily-sales-report-generation
  - id: weekly-database-maintenance  
  - id: real-time-order-processing-webhook

  # Avoid - generic and unclear
  - id: trigger1
  - id: schedule
  - id: webhook
```

## Next Steps

- Learn about [Best Practices](best-practices.md) for maintainable workflows
- Review [Error Handling](error-handling.md) for robust automation
- Explore [Flow Control](flow-control.md) for complex execution patterns