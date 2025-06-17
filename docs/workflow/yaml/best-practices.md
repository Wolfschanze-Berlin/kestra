# Best Practices

This guide provides comprehensive best practices for creating maintainable, efficient, and robust YAML workflows in Kestra.

## Workflow Design Principles

### 1. Single Responsibility Principle

Design workflows with a clear, single purpose. Avoid creating monolithic workflows that handle multiple unrelated concerns.

#### Good Example
```yaml
id: user-data-etl
namespace: analytics
description: "Extract, transform, and load user data from CRM to data warehouse"

tasks:
  - id: extract-users
    type: io.kestra.plugin.core.log.Log
    message: "Extracting user data from CRM"
    
  - id: transform-users
    type: io.kestra.plugin.core.log.Log
    message: "Transforming user data for warehouse"
    
  - id: load-users
    type: io.kestra.plugin.core.log.Log
    message: "Loading transformed user data"
```

#### Avoid
```yaml
id: everything-workflow
namespace: analytics
description: "Does everything: ETL, reporting, monitoring, cleanup, and more"

tasks:
  - id: extract-users
  - id: generate-reports
  - id: send-emails
  - id: cleanup-logs
  - id: update-inventory
  # ... too many unrelated responsibilities
```

### 2. Descriptive Naming Conventions

Use clear, descriptive names for workflows, tasks, and variables that explain their purpose.

#### Naming Guidelines
```yaml
# Workflow IDs: kebab-case with domain context
id: customer-data-pipeline
id: inventory-reconciliation-process
id: security-audit-report

# Task IDs: action-oriented with kebab-case
tasks:
  - id: validate-input-data
  - id: transform-customer-records
  - id: load-to-warehouse
  - id: send-completion-notification

# Variable names: snake_case for readability
variables:
  database_connection_timeout: 30
  max_retry_attempts: 3
  email_notification_enabled: true
```

### 3. Modular Workflow Design

Break complex workflows into smaller, reusable components.

#### Parent Workflow
```yaml
id: data-pipeline-orchestrator
namespace: analytics

tasks:
  - id: run-extraction
    type: io.kestra.plugin.core.flow.Subflow
    namespace: analytics
    flowId: data-extraction-pipeline
    inputs:
      source_system: "crm"
      extraction_date: "{{ now() | date('yyyy-MM-dd') }}"
      
  - id: run-transformation
    type: io.kestra.plugin.core.flow.Subflow
    namespace: analytics
    flowId: data-transformation-pipeline
    inputs:
      input_data: "{{ outputs.run-extraction.outputs.extracted_data }}"
      
  - id: run-loading
    type: io.kestra.plugin.core.flow.Subflow
    namespace: analytics
    flowId: data-loading-pipeline
    inputs:
      transformed_data: "{{ outputs.run-transformation.outputs.transformed_data }}"
```

#### Reusable Subflows
```yaml
id: data-extraction-pipeline
namespace: analytics
description: "Reusable data extraction component"

inputs:
  - id: source_system
    type: STRING
    required: true
  - id: extraction_date
    type: STRING
    required: true

tasks:
  - id: extract-data
    type: io.kestra.plugin.core.debug.Return
    format: |
      {
        "source": "{{ inputs.source_system }}",
        "date": "{{ inputs.extraction_date }}",
        "records": 10000
      }

outputs:
  - id: extracted_data
    type: JSON
    value: "{{ outputs.extract-data.value }}"
```

## Code Organization

### 1. Consistent Structure

Organize workflow elements in a consistent order:

```yaml
id: example-workflow
namespace: company.team
description: "Clear description of what this workflow does"

labels:
  team: data-engineering
  environment: production
  criticality: high

inputs:
  - id: input1
    type: STRING
    required: true
    description: "Description of input"

variables:
  config_value: "some_value"
  timeout_seconds: 300

tasks:
  - id: first-task
    type: io.kestra.plugin.core.log.Log
    message: "Task execution"

finally:
  - id: cleanup-task
    type: io.kestra.plugin.core.log.Log
    message: "Cleanup operations"

errors:
  - id: error-handler
    type: io.kestra.plugin.core.log.Log
    message: "Error handling"

outputs:
  - id: result
    type: STRING
    value: "{{ outputs.first-task.value }}"

triggers:
  - id: schedule-trigger
    type: io.kestra.plugin.core.trigger.Schedule
    cron: "0 9 * * *"
```

### 2. Documentation and Comments

Provide comprehensive documentation within your workflows.

```yaml
id: well-documented-workflow
namespace: analytics
description: |
  ## Customer Data Processing Pipeline
  
  This workflow processes daily customer data from multiple sources:
  
  1. **Extraction**: Pulls data from CRM and website analytics
  2. **Validation**: Ensures data quality and completeness
  3. **Transformation**: Standardizes formats and calculates metrics
  4. **Loading**: Stores processed data in the data warehouse
  
  **Dependencies**: Requires CRM system and analytics API to be available
  **Runtime**: Approximately 30-45 minutes
  **Output**: Updated customer dimension tables

labels:
  team: data-engineering
  owner: data-team
  project: customer-analytics
  environment: production
  sla: "2-hours"

variables:
  # Database configuration
  warehouse_connection:
    host: "warehouse.company.com"
    database: "analytics"
    timeout: 300
    
  # Processing configuration  
  batch_size: 1000
  max_retries: 3
  
  # Quality thresholds
  min_data_quality_score: 0.95
  max_null_percentage: 5.0

tasks:
  - id: validate-prerequisites
    type: io.kestra.plugin.core.log.Log
    description: |
      Validates that all required systems are available before processing.
      Checks CRM API connectivity and warehouse accessibility.
    message: "Validating system prerequisites"
    
  - id: extract-crm-data
    type: io.kestra.plugin.core.log.Log
    description: |
      Extracts customer data from CRM system using REST API.
      Retrieves customers modified in the last 24 hours.
    message: "Extracting CRM data for {{ now() | date('yyyy-MM-dd') }}"
```

### 3. Environment Configuration

Use variables and inputs to make workflows environment-agnostic.

```yaml
id: environment-agnostic-workflow
namespace: company.team

inputs:
  - id: environment
    type: SELECT
    values: [development, staging, production]
    defaults: development
    description: "Target environment for deployment"
    
  - id: debug_mode
    type: BOOL
    defaults: false
    description: "Enable detailed debug logging"

variables:
  # Environment-specific configuration
  database_host: "{{ inputs.environment == 'production' ? 'prod-db.company.com' : 'test-db.company.com' }}"
  batch_size: "{{ inputs.environment == 'production' ? 5000 : 100 }}"
  timeout_minutes: "{{ inputs.environment == 'production' ? 60 : 10 }}"
  
  # Feature flags
  enable_notifications: "{{ inputs.environment == 'production' }}"
  enable_monitoring: "{{ inputs.environment != 'development' }}"
  
  # Logging configuration
  log_level: "{{ inputs.debug_mode ? 'DEBUG' : (inputs.environment == 'production' ? 'INFO' : 'DEBUG') }}"

tasks:
  - id: environment-setup
    type: io.kestra.plugin.core.log.Log
    message: |
      Environment Configuration:
      - Environment: {{ inputs.environment }}
      - Database: {{ vars.database_host }}
      - Batch Size: {{ render(vars.batch_size) }}
      - Timeout: {{ render(vars.timeout_minutes) }} minutes
      - Debug Mode: {{ inputs.debug_mode }}
      - Log Level: {{ render(vars.log_level) }}
```

## Error Handling Best Practices

### 1. Layered Error Handling

Implement error handling at multiple levels: task, workflow, and system.

```yaml
id: layered-error-handling
namespace: company.team

# Workflow-level retry for transient failures
retry:
  behavior: RETRY_FAILED_TASK
  type: exponential
  interval: PT1M
  maxAttempt: 3

tasks:
  - id: critical-database-operation
    type: io.kestra.plugin.core.log.Log
    message: "Performing critical database operation"
    # Task-level retry for immediate recovery
    retry:
      type: constant
      interval: PT30S
      maxAttempt: 3
    # Don't fail workflow for non-critical operations
    allowFailure: false
    
  - id: optional-notification
    type: io.kestra.plugin.core.log.Log
    message: "Sending optional notification"
    allowFailure: true  # Don't fail workflow if notification fails
    retry:
      type: constant
      interval: PT10S
      maxAttempt: 2

# Workflow-level error handling
errors:
  - id: log-error-context
    type: io.kestra.plugin.core.log.Log
    message: |
      Error occurred in workflow {{ flow.id }}:
      - Execution ID: {{ execution.id }}
      - Failed at: {{ now() }}
      - Error details: {{ errorLogs() | truncate(500) }}
      
  - id: cleanup-on-error
    type: io.kestra.plugin.core.log.Log
    message: "Performing cleanup operations"
    
  - id: send-error-notification
    type: io.kestra.plugin.core.log.Log
    message: "Sending error notification to operations team"
    allowFailure: true

finally:
  - id: always-cleanup
    type: io.kestra.plugin.core.log.Log
    message: "Executing cleanup regardless of success or failure"
```

### 2. Graceful Degradation

Design workflows to handle partial failures gracefully.

```yaml
id: graceful-degradation-example
namespace: company.team

tasks:
  - id: primary-data-source
    type: io.kestra.plugin.core.debug.Return
    format: '{"status": "success", "records": 1000}'
    allowFailure: true
    retry:
      type: constant
      interval: PT15S
      maxAttempt: 2
      
  - id: fallback-processing
    type: io.kestra.plugin.core.flow.If
    condition: "{{ outputs.primary-data-source.value == null }}"
    then:
      - id: use-backup-source
        type: io.kestra.plugin.core.debug.Return
        format: '{"status": "degraded", "records": 500}'
        
      - id: log-degraded-mode
        type: io.kestra.plugin.core.log.Log
        message: "⚠️ Operating in degraded mode - using backup data source"
        
  - id: process-available-data
    type: io.kestra.plugin.core.log.Log
    message: |
      Processing data:
      - Source: {{ outputs.primary-data-source.value != null ? 'primary' : 'backup' }}
      - Records: {{ outputs.primary-data-source.value != null ? fromJson(outputs.primary-data-source.value).records : fromJson(outputs.use-backup-source.value).records }}
      - Quality: {{ outputs.primary-data-source.value != null ? 'high' : 'reduced' }}
```

## Performance Optimization

### 1. Parallel Processing

Use parallel execution for independent operations.

```yaml
id: performance-optimized-workflow
namespace: company.team

tasks:
  - id: parallel-data-processing
    type: io.kestra.plugin.core.flow.Parallel
    concurrent: 4  # Limit concurrent tasks to prevent resource exhaustion
    tasks:
      - id: process-region-us
        type: io.kestra.plugin.core.log.Log
        message: "Processing US region data"
        
      - id: process-region-eu
        type: io.kestra.plugin.core.log.Log
        message: "Processing EU region data"
        
      - id: process-region-asia
        type: io.kestra.plugin.core.log.Log
        message: "Processing Asia region data"
        
      - id: process-region-latam
        type: io.kestra.plugin.core.log.Log
        message: "Processing Latin America region data"
        
  - id: aggregate-results
    type: io.kestra.plugin.core.log.Log
    message: "Aggregating results from all regions"
```

### 2. Efficient Data Handling

Minimize data transfer and processing overhead.

```yaml
id: efficient-data-handling
namespace: company.team

variables:
  # Use appropriate batch sizes
  batch_size: 5000
  # Configure reasonable timeouts
  processing_timeout: "PT30M"
  # Enable compression for large data transfers
  enable_compression: true

tasks:
  - id: stream-large-dataset
    type: io.kestra.plugin.core.log.Log
    message: "Processing data in batches of {{ vars.batch_size }}"
    timeout: "{{ vars.processing_timeout }}"
    
  - id: process-in-chunks
    type: io.kestra.plugin.core.flow.EachParallel
    value: "{{ range(0, 10) }}"  # Process 10 chunks in parallel
    concurrent: 3  # Limit concurrent processing
    tasks:
      - id: process-chunk
        type: io.kestra.plugin.core.log.Log
        message: "Processing chunk {{ taskrun.iteration }} with batch size {{ vars.batch_size }}"
```

## Security Best Practices

### 1. Secure Secret Management

Never hardcode secrets in workflows; use Kestra's secret management.

```yaml
id: secure-workflow
namespace: company.team

variables:
  # Use secrets for sensitive data
  database_host: "{{ secret('DATABASE_HOST') }}"
  api_endpoint: "https://api.example.com"
  
  # Use environment-specific secrets
  database_password: "{{ secret('DB_PASSWORD_' + inputs.environment | upper) }}"

tasks:
  - id: secure-database-connection
    type: io.kestra.plugin.core.log.Log
    message: "Connecting to database at {{ vars.database_host }}"
    # Never log sensitive information
    
  - id: api-call-with-auth
    type: io.kestra.plugin.core.log.Log
    message: "Making authenticated API call"
    # Use secret for API keys
    # api_key: "{{ secret('API_KEY') }}"
```

### 2. Input Validation and Sanitization

Validate and sanitize all inputs to prevent security issues.

```yaml
id: input-validation-workflow
namespace: company.team

inputs:
  - id: email
    type: STRING
    required: true
    description: "User email address"
    
  - id: file_path
    type: STRING
    required: true
    description: "File path for processing"

tasks:
  - id: validate-email-format
    type: io.kestra.plugin.core.flow.If
    condition: "{{ inputs.email | match('^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$') }}"
    then:
      - id: process-valid-email
        type: io.kestra.plugin.core.log.Log
        message: "Processing valid email: {{ inputs.email }}"
    else:
      - id: reject-invalid-email
        type: io.kestra.plugin.core.execution.Fail
        message: "Invalid email format: {{ inputs.email }}"
        
  - id: validate-file-path
    type: io.kestra.plugin.core.flow.If
    condition: "{{ !inputs.file_path.contains('..') and inputs.file_path.startsWith('/allowed/path/') }}"
    then:
      - id: process-safe-path
        type: io.kestra.plugin.core.log.Log
        message: "Processing file at safe path: {{ inputs.file_path }}"
    else:
      - id: reject-unsafe-path
        type: io.kestra.plugin.core.execution.Fail
        message: "Unsafe file path detected: {{ inputs.file_path }}"
```

## Monitoring and Observability

### 1. Comprehensive Logging

Implement structured logging for observability.

```yaml
id: observable-workflow
namespace: company.team

variables:
  correlation_id: "{{ execution.id }}"
  workflow_version: "2.1.0"

tasks:
  - id: log-workflow-start
    type: io.kestra.plugin.core.log.Log
    message: |
      {
        "event": "workflow_started",
        "correlation_id": "{{ vars.correlation_id }}",
        "workflow_id": "{{ flow.id }}",
        "version": "{{ vars.workflow_version }}",
        "timestamp": "{{ now() }}",
        "environment": "{{ inputs.environment | default('unknown') }}"
      }
      
  - id: business-logic-task
    type: io.kestra.plugin.core.log.Log
    message: |
      {
        "event": "processing_data",
        "correlation_id": "{{ vars.correlation_id }}",
        "task_id": "{{ task.id }}",
        "timestamp": "{{ now() }}",
        "status": "in_progress"
      }
      
  - id: log-workflow-completion
    type: io.kestra.plugin.core.log.Log
    message: |
      {
        "event": "workflow_completed",
        "correlation_id": "{{ vars.correlation_id }}",
        "workflow_id": "{{ flow.id }}",
        "duration_seconds": {{ (now() | timestamp) - (execution.startDate | timestamp) }},
        "timestamp": "{{ now() }}",
        "status": "success"
      }
```

### 2. Health Checks and Monitoring

Include health checks and monitoring tasks.

```yaml
id: self-monitoring-workflow
namespace: company.team

tasks:
  - id: health-check
    type: io.kestra.plugin.core.log.Log
    message: "Performing system health check"
    
  - id: check-dependencies
    type: io.kestra.plugin.core.flow.Parallel
    tasks:
      - id: check-database
        type: io.kestra.plugin.core.log.Log
        message: "Database connectivity: OK"
        
      - id: check-api-endpoints
        type: io.kestra.plugin.core.log.Log
        message: "API endpoints: OK"
        
      - id: check-storage
        type: io.kestra.plugin.core.log.Log
        message: "Storage access: OK"
        
  - id: main-processing
    type: io.kestra.plugin.core.log.Log
    message: "Executing main business logic"
    
  - id: performance-metrics
    type: io.kestra.plugin.core.debug.Return
    format: |
      {
        "execution_time_seconds": {{ (now() | timestamp) - (execution.startDate | timestamp) }},
        "memory_usage": "128MB",
        "records_processed": 10000,
        "throughput_per_second": {{ 10000 / ((now() | timestamp) - (execution.startDate | timestamp)) | round(2) }}
      }

outputs:
  - id: performance_metrics
    type: JSON
    value: "{{ outputs.performance-metrics.value }}"
```

## Testing and Validation

### 1. Built-in Validation

Include validation tasks within workflows.

```yaml
id: self-validating-workflow
namespace: company.team

inputs:
  - id: data_quality_threshold
    type: FLOAT
    defaults: 0.95
    description: "Minimum acceptable data quality score"

tasks:
  - id: process-data
    type: io.kestra.plugin.core.debug.Return
    format: |
      {
        "processed_records": 10000,
        "failed_records": 50,
        "quality_score": 0.995
      }
      
  - id: validate-data-quality
    type: io.kestra.plugin.core.flow.If
    condition: "{{ fromJson(outputs.process-data.value).quality_score >= inputs.data_quality_threshold }}"
    then:
      - id: quality-check-passed
        type: io.kestra.plugin.core.log.Log
        message: "✅ Data quality check passed ({{ fromJson(outputs.process-data.value).quality_score }})"
    else:
      - id: quality-check-failed
        type: io.kestra.plugin.core.execution.Fail
        message: "❌ Data quality below threshold: {{ fromJson(outputs.process-data.value).quality_score }} < {{ inputs.data_quality_threshold }}"
        
  - id: validate-record-counts
    type: io.kestra.plugin.core.flow.If
    condition: "{{ fromJson(outputs.process-data.value).processed_records > 0 }}"
    then:
      - id: records-validated
        type: io.kestra.plugin.core.log.Log
        message: "✅ Record count validation passed"
    else:
      - id: no-records-error
        type: io.kestra.plugin.core.execution.Fail
        message: "❌ No records processed - potential data source issue"
```

### 2. Test Data Patterns

Use test data for development and testing.

```yaml
id: testable-workflow
namespace: company.team

inputs:
  - id: use_test_data
    type: BOOL
    defaults: false
    description: "Use test data instead of production data"

variables:
  test_data: |
    [
      {"id": 1, "name": "Test User 1", "email": "test1@example.com"},
      {"id": 2, "name": "Test User 2", "email": "test2@example.com"}
    ]

tasks:
  - id: get-data-source
    type: io.kestra.plugin.core.flow.If
    condition: "{{ inputs.use_test_data }}"
    then:
      - id: use-test-data
        type: io.kestra.plugin.core.debug.Return
        format: "{{ vars.test_data }}"
    else:
      - id: use-production-data
        type: io.kestra.plugin.core.log.Log
        message: "Fetching data from production source"
        
  - id: process-data
    type: io.kestra.plugin.core.log.Log
    message: |
      Processing {{ inputs.use_test_data ? 'test' : 'production' }} data
      {% if inputs.use_test_data -%}
      Records: {{ fromJson(outputs.use-test-data.value) | length }}
      {% endif %}
```

## Maintenance and Lifecycle

### 1. Version Management

Track workflow versions and changes.

```yaml
id: versioned-workflow
namespace: company.team
description: |
  Customer Data Pipeline v3.2.1
  
  ## Changelog
  - v3.2.1: Fixed data validation edge case
  - v3.2.0: Added new customer attributes
  - v3.1.0: Improved error handling
  - v3.0.0: Major refactoring for performance

labels:
  version: "3.2.1"
  last_updated: "2024-01-15"
  maintainer: "data-team"
  
variables:
  pipeline_version: "3.2.1"
  compatible_api_version: "2.x"

tasks:
  - id: log-version-info
    type: io.kestra.plugin.core.log.Log
    message: |
      Pipeline Version Information:
      - Version: {{ vars.pipeline_version }}
      - Compatible API: {{ vars.compatible_api_version }}
      - Flow Revision: {{ flow.revision }}
```

### 2. Deprecation Handling

Gracefully handle deprecated features and migrations.

```yaml
id: migration-aware-workflow
namespace: company.team

inputs:
  - id: use_legacy_api
    type: BOOL
    defaults: false
    description: "Use legacy API (deprecated, will be removed in v4.0)"

tasks:
  - id: deprecation-warning
    type: io.kestra.plugin.core.log.Log
    message: "⚠️ WARNING: Legacy API usage detected. Please migrate to new API before v4.0"
    runIf: "{{ inputs.use_legacy_api }}"
    
  - id: api-call
    type: io.kestra.plugin.core.flow.If
    condition: "{{ inputs.use_legacy_api }}"
    then:
      - id: legacy-api-call
        type: io.kestra.plugin.core.log.Log
        message: "Using deprecated legacy API"
    else:
      - id: modern-api-call
        type: io.kestra.plugin.core.log.Log
        message: "Using modern API v2"
```

## Documentation Standards

### 1. Comprehensive Flow Documentation

```yaml
id: well-documented-example
namespace: company.team
description: |
  # Customer Data Synchronization Pipeline
  
  ## Purpose
  Synchronizes customer data between CRM system and data warehouse on a daily basis.
  
  ## Dependencies
  - CRM API (requires authentication token)
  - Data warehouse database connection
  - Email service for notifications
  
  ## Inputs
  - `sync_date`: Date to sync (defaults to yesterday)
  - `dry_run`: Preview mode without making changes
  
  ## Outputs
  - `sync_summary`: JSON summary of synchronization results
  - `record_count`: Number of records processed
  
  ## Schedule
  Runs daily at 2:00 AM UTC via scheduled trigger
  
  ## Error Handling
  - Retries transient failures up to 3 times
  - Sends notifications on persistent failures
  - Continues with partial data if non-critical systems fail
  
  ## Performance
  - Typical runtime: 15-30 minutes
  - Processes ~100K records per run
  - Uses parallel processing for large datasets
  
  ## Monitoring
  - Logs structured JSON for observability
  - Tracks data quality metrics
  - Alerts on SLA violations

labels:
  team: data-engineering
  criticality: high
  sla_hours: 4
  documentation_url: "https://wiki.company.com/data-sync-pipeline"
```

## Conclusion

Following these best practices will help you create workflows that are:

- **Maintainable**: Easy to understand, modify, and debug
- **Reliable**: Handle errors gracefully and recover from failures
- **Performant**: Execute efficiently with appropriate resource usage
- **Secure**: Protect sensitive data and validate inputs
- **Observable**: Provide comprehensive logging and monitoring
- **Testable**: Include validation and support test scenarios

Remember that best practices evolve with your organization's needs and the Kestra platform capabilities. Regularly review and update your workflows to incorporate new features and improved patterns.

## Related Documentation

- [Basic Workflows](basic-workflows.md) - Fundamentals of workflow creation
- [Error Handling](error-handling.md) - Comprehensive error handling strategies
- [Flow Control](flow-control.md) - Advanced execution patterns
- [Variables and Data](variables-and-data.md) - Data handling and templating
- [Triggers and Scheduling](triggers-scheduling.md) - Automated execution patterns