# Flow Control

This guide covers how to control the execution flow of your workflows using sequential, parallel, and conditional execution patterns.

## Sequential Execution (Default)

By default, tasks in Kestra execute sequentially - one after another in the order they are defined.

### Basic Sequential Flow

```yaml
id: sequential-example
namespace: tutorial
description: "Tasks execute one after another"

tasks:
  - id: step-1
    type: io.kestra.plugin.core.log.Log
    message: "Step 1: Initialize"
    
  - id: step-2
    type: io.kestra.plugin.core.log.Log
    message: "Step 2: Process (depends on step-1 completion)"
    
  - id: step-3
    type: io.kestra.plugin.core.log.Log
    message: "Step 3: Finalize (depends on step-2 completion)"
```

### Sequential with Data Dependencies

```yaml
id: sequential-data-flow
namespace: tutorial

tasks:
  - id: extract
    type: io.kestra.plugin.core.debug.Return
    format: |
      {
        "records": [
          {"id": 1, "name": "Alice"},
          {"id": 2, "name": "Bob"}
        ],
        "count": 2
      }
      
  - id: transform
    type: io.kestra.plugin.core.debug.Return
    format: |
      {
        "processed_records": {{ fromJson(outputs.extract.value).records | length }},
        "transformation": "uppercase_names",
        "timestamp": "{{ now() }}"
      }
      
  - id: load
    type: io.kestra.plugin.core.log.Log
    message: |
      Loading {{ fromJson(outputs.transform.value).processed_records }} records
      Transformation: {{ fromJson(outputs.transform.value).transformation }}
      At: {{ fromJson(outputs.transform.value).timestamp }}
```

## Parallel Execution

Use the `Parallel` task to execute multiple tasks simultaneously, improving performance for independent operations.

### Basic Parallel Execution

```yaml
id: parallel-basic-example
namespace: tutorial

tasks:
  - id: setup
    type: io.kestra.plugin.core.log.Log
    message: "Setting up parallel processing"
    
  - id: parallel-processing
    type: io.kestra.plugin.core.flow.Parallel
    tasks:
      - id: task-a
        type: io.kestra.plugin.core.log.Log
        message: "Processing dataset A"
        
      - id: task-b
        type: io.kestra.plugin.core.log.Log
        message: "Processing dataset B"
        
      - id: task-c
        type: io.kestra.plugin.core.log.Log
        message: "Processing dataset C"
        
  - id: cleanup
    type: io.kestra.plugin.core.log.Log
    message: "Cleaning up after parallel processing"
```

### Parallel with Concurrency Limits

```yaml
id: parallel-with-limits
namespace: tutorial
description: "Limit concurrent execution to prevent resource exhaustion"

tasks:
  - id: concurrent-processing
    type: io.kestra.plugin.core.flow.Parallel
    concurrent: 2  # Only run 2 tasks at a time
    tasks:
      - id: heavy-task-1
        type: io.kestra.plugin.core.log.Log
        message: "Heavy processing task 1"
        
      - id: heavy-task-2
        type: io.kestra.plugin.core.log.Log
        message: "Heavy processing task 2"
        
      - id: heavy-task-3
        type: io.kestra.plugin.core.log.Log
        message: "Heavy processing task 3"
        
      - id: heavy-task-4
        type: io.kestra.plugin.core.log.Log
        message: "Heavy processing task 4"
```

### Nested Parallel Execution

```yaml
id: nested-parallel-example
namespace: tutorial

tasks:
  - id: main-parallel
    type: io.kestra.plugin.core.flow.Parallel
    tasks:
      - id: data-pipeline-a
        type: io.kestra.plugin.core.flow.Parallel
        tasks:
          - id: extract-a
            type: io.kestra.plugin.core.log.Log
            message: "Extracting data A"
          - id: validate-a
            type: io.kestra.plugin.core.log.Log
            message: "Validating data A"
            
      - id: data-pipeline-b
        type: io.kestra.plugin.core.flow.Parallel
        tasks:
          - id: extract-b
            type: io.kestra.plugin.core.log.Log
            message: "Extracting data B"
          - id: validate-b
            type: io.kestra.plugin.core.log.Log
            message: "Validating data B"
```

## Conditional Execution

Control which tasks execute based on conditions using the `runIf` property or `If` task.

### Using runIf Property

```yaml
id: conditional-runif-example
namespace: tutorial

inputs:
  - id: environment
    type: SELECT
    values:
      - development
      - staging
      - production
    defaults: development
    
  - id: enable_notifications
    type: BOOL
    defaults: false

tasks:
  - id: setup
    type: io.kestra.plugin.core.log.Log
    message: "Setting up for {{ inputs.environment }} environment"
    
  - id: dev-setup
    type: io.kestra.plugin.core.log.Log
    message: "Configuring development settings"
    runIf: "{{ inputs.environment == 'development' }}"
    
  - id: prod-setup
    type: io.kestra.plugin.core.log.Log
    message: "Configuring production settings with enhanced security"
    runIf: "{{ inputs.environment == 'production' }}"
    
  - id: send-notification
    type: io.kestra.plugin.core.log.Log
    message: "Sending notification for {{ inputs.environment }} deployment"
    runIf: "{{ inputs.enable_notifications }}"
    
  - id: finalize
    type: io.kestra.plugin.core.log.Log
    message: "Setup complete"
```

### Using If Task

```yaml
id: conditional-if-task-example
namespace: tutorial

inputs:
  - id: data_size
    type: INT
    defaults: 1000

tasks:
  - id: check-data-size
    type: io.kestra.plugin.core.flow.If
    condition: "{{ inputs.data_size > 5000 }}"
    then:
      - id: large-data-processing
        type: io.kestra.plugin.core.log.Log
        message: "Processing large dataset ({{ inputs.data_size }} records) with optimized strategy"
        
      - id: enable-parallelization
        type: io.kestra.plugin.core.log.Log
        message: "Enabling parallel processing for large dataset"
    else:
      - id: small-data-processing
        type: io.kestra.plugin.core.log.Log
        message: "Processing small dataset ({{ inputs.data_size }} records) with standard strategy"
```

### Complex Conditional Logic

```yaml
id: complex-conditional-example
namespace: tutorial

inputs:
  - id: data_source
    type: SELECT
    values: [database, api, file]
    defaults: database
    
  - id: is_urgent
    type: BOOL
    defaults: false
    
  - id: retry_count
    type: INT
    defaults: 0

variables:
  max_retries: 3

tasks:
  - id: validate-inputs
    type: io.kestra.plugin.core.log.Log
    message: "Validating inputs: source={{ inputs.data_source }}, urgent={{ inputs.is_urgent }}"
    
  - id: urgent-processing
    type: io.kestra.plugin.core.flow.If
    condition: "{{ inputs.is_urgent }}"
    then:
      - id: priority-handler
        type: io.kestra.plugin.core.log.Log
        message: "🚨 URGENT: Fast-tracking processing for {{ inputs.data_source }}"
        
      - id: skip-validation
        type: io.kestra.plugin.core.log.Log
        message: "Skipping extended validation due to urgency"
    else:
      - id: standard-validation
        type: io.kestra.plugin.core.log.Log
        message: "Performing standard validation"
        
  - id: source-specific-logic
    type: io.kestra.plugin.core.flow.If
    condition: "{{ inputs.data_source == 'api' }}"
    then:
      - id: api-processing
        type: io.kestra.plugin.core.log.Log
        message: "Processing API data"
        
      - id: rate-limit-check
        type: io.kestra.plugin.core.log.Log
        message: "Checking API rate limits"
    else:
      - id: file-or-db-processing
        type: io.kestra.plugin.core.flow.If
        condition: "{{ inputs.data_source == 'file' }}"
        then:
          - id: file-processing
            type: io.kestra.plugin.core.log.Log
            message: "Processing file data"
        else:
          - id: database-processing
            type: io.kestra.plugin.core.log.Log
            message: "Processing database data"
            
  - id: retry-logic
    type: io.kestra.plugin.core.log.Log
    message: "Retry {{ inputs.retry_count }}/{{ vars.max_retries }}"
    runIf: "{{ inputs.retry_count > 0 && inputs.retry_count <= vars.max_retries }}"
```

## Iteration Patterns

### Sequential Processing with Each

```yaml
id: sequential-each-example
namespace: tutorial

inputs:
  - id: items
    type: JSON
    defaults: |
      [
        {"id": 1, "name": "item1"},
        {"id": 2, "name": "item2"},
        {"id": 3, "name": "item3"}
      ]

tasks:
  - id: process-each-item
    type: io.kestra.plugin.core.flow.EachSequential
    value: "{{ inputs.items }}"
    tasks:
      - id: validate-item
        type: io.kestra.plugin.core.log.Log
        message: "Validating item {{ taskrun.value.id }}: {{ taskrun.value.name }}"
        
      - id: process-item
        type: io.kestra.plugin.core.debug.Return
        format: |
          {
            "original": {{ taskrun.value | toJson }},
            "processed_at": "{{ now() }}",
            "iteration": {{ taskrun.iteration }}
          }
```

### Parallel Processing with Each

```yaml
id: parallel-each-example
namespace: tutorial

inputs:
  - id: datasets
    type: JSON
    defaults: |
      [
        {"name": "dataset_a", "size": 1000},
        {"name": "dataset_b", "size": 2000},
        {"name": "dataset_c", "size": 1500}
      ]

tasks:
  - id: process-datasets-parallel
    type: io.kestra.plugin.core.flow.EachParallel
    value: "{{ inputs.datasets }}"
    concurrent: 2
    tasks:
      - id: analyze-dataset
        type: io.kestra.plugin.core.log.Log
        message: "Analyzing {{ taskrun.value.name }} ({{ taskrun.value.size }} records)"
        
      - id: generate-report
        type: io.kestra.plugin.core.debug.Return
        format: |
          {
            "dataset": "{{ taskrun.value.name }}",
            "analysis_complete": true,
            "processed_at": "{{ now() }}"
          }
```

## Switch/Case Pattern

```yaml
id: switch-case-example
namespace: tutorial

inputs:
  - id: operation_type
    type: SELECT
    values: [create, update, delete, read]
    defaults: read

tasks:
  - id: operation-router
    type: io.kestra.plugin.core.flow.Switch
    value: "{{ inputs.operation_type }}"
    cases:
      create:
        - id: create-operation
          type: io.kestra.plugin.core.log.Log
          message: "Performing CREATE operation"
          
        - id: validate-creation
          type: io.kestra.plugin.core.log.Log
          message: "Validating created resource"
          
      update:
        - id: update-operation
          type: io.kestra.plugin.core.log.Log
          message: "Performing UPDATE operation"
          
        - id: validate-update
          type: io.kestra.plugin.core.log.Log
          message: "Validating updated resource"
          
      delete:
        - id: backup-before-delete
          type: io.kestra.plugin.core.log.Log
          message: "Creating backup before deletion"
          
        - id: delete-operation
          type: io.kestra.plugin.core.log.Log
          message: "Performing DELETE operation"
          
    defaults:
      - id: read-operation
        type: io.kestra.plugin.core.log.Log
        message: "Performing READ operation (default)"
```

## Advanced Flow Control Patterns

### Pipeline Pattern

```yaml
id: pipeline-pattern
namespace: tutorial
description: "ETL pipeline with conditional stages"

inputs:
  - id: skip_validation
    type: BOOL
    defaults: false

tasks:
  # Stage 1: Extract
  - id: extract-stage
    type: io.kestra.plugin.core.flow.Parallel
    tasks:
      - id: extract-source-a
        type: io.kestra.plugin.core.debug.Return
        format: '{"source": "A", "records": 100}'
        
      - id: extract-source-b
        type: io.kestra.plugin.core.debug.Return
        format: '{"source": "B", "records": 150}'
  
  # Stage 2: Validate (conditional)
  - id: validation-stage
    type: io.kestra.plugin.core.flow.If
    condition: "{{ !inputs.skip_validation }}"
    then:
      - id: validate-source-a
        type: io.kestra.plugin.core.log.Log
        message: "Validating {{ fromJson(outputs.extract-source-a.value).records }} records from source A"
        
      - id: validate-source-b
        type: io.kestra.plugin.core.log.Log
        message: "Validating {{ fromJson(outputs.extract-source-b.value).records }} records from source B"
  
  # Stage 3: Transform
  - id: transform-stage
    type: io.kestra.plugin.core.flow.Parallel
    tasks:
      - id: transform-a
        type: io.kestra.plugin.core.debug.Return
        format: |
          {
            "source": "A",
            "original_count": {{ fromJson(outputs.extract-source-a.value).records }},
            "transformed_count": {{ fromJson(outputs.extract-source-a.value).records * 2 }},
            "validation_skipped": {{ inputs.skip_validation }}
          }
          
      - id: transform-b
        type: io.kestra.plugin.core.debug.Return
        format: |
          {
            "source": "B",
            "original_count": {{ fromJson(outputs.extract-source-b.value).records }},
            "transformed_count": {{ fromJson(outputs.extract-source-b.value).records * 2 }},
            "validation_skipped": {{ inputs.skip_validation }}
          }
  
  # Stage 4: Load
  - id: load-stage
    type: io.kestra.plugin.core.log.Log
    message: |
      Loading transformed data:
      - Source A: {{ fromJson(outputs.transform-a.value).transformed_count }} records
      - Source B: {{ fromJson(outputs.transform-b.value).transformed_count }} records
      - Total: {{ fromJson(outputs.transform-a.value).transformed_count + fromJson(outputs.transform-b.value).transformed_count }} records
```

### Fan-out/Fan-in Pattern

```yaml
id: fan-out-fan-in-pattern
namespace: tutorial

tasks:
  - id: prepare-data
    type: io.kestra.plugin.core.debug.Return
    format: |
      {
        "dataset": "main_data",
        "partitions": ["part_1", "part_2", "part_3", "part_4"]
      }
  
  # Fan-out: Process each partition in parallel
  - id: fan-out-processing
    type: io.kestra.plugin.core.flow.EachParallel
    value: "{{ fromJson(outputs.prepare-data.value).partitions }}"
    concurrent: 4
    tasks:
      - id: process-partition
        type: io.kestra.plugin.core.debug.Return
        format: |
          {
            "partition": "{{ taskrun.value }}",
            "processed_records": {{ random() * 1000 | round }},
            "processing_time": "{{ random() * 60 | round }}s"
          }
  
  # Fan-in: Aggregate results
  - id: fan-in-aggregation
    type: io.kestra.plugin.core.debug.Return
    format: |
      {
        "total_partitions": {{ outputs.fan-out-processing.value | length }},
        "aggregated_results": {{ outputs.fan-out-processing.value | toJson }},
        "completion_time": "{{ now() }}"
      }
      
  - id: summary
    type: io.kestra.plugin.core.log.Log
    message: |
      Processing complete:
      - Processed {{ fromJson(outputs.fan-in-aggregation.value).total_partitions }} partitions
      - Completed at {{ fromJson(outputs.fan-in-aggregation.value).completion_time }}
```

## Best Practices

### 1. Use Descriptive Task IDs
```yaml
# Good
- id: validate-input-data
- id: transform-customer-records
- id: load-to-warehouse

# Avoid
- id: task1
- id: step2
- id: process
```

### 2. Group Related Operations
```yaml
# Use parallel for independent operations
- id: data-preprocessing
  type: io.kestra.plugin.core.flow.Parallel
  tasks:
    - id: validate-schema
    - id: check-data-quality
    - id: calculate-statistics
```

### 3. Implement Proper Error Boundaries
```yaml
# Use allowFailure for non-critical tasks
- id: send-notification
  type: io.kestra.plugin.core.log.Log
  message: "Processing complete"
  allowFailure: true  # Don't fail workflow if notification fails
```

### 4. Use Conditions Effectively
```yaml
# Use runIf for simple conditions
- id: cleanup-temp-files
  type: io.kestra.plugin.core.log.Log
  message: "Cleaning up temporary files"
  runIf: "{{ inputs.cleanup_enabled | default(true) }}"

# Use If task for complex conditional logic
- id: complex-decision
  type: io.kestra.plugin.core.flow.If
  condition: "{{ inputs.data_size > 1000 && inputs.processing_mode == 'batch' }}"
```

## Next Steps

- Learn about [Variables and Data](variables-and-data.md) for advanced data handling
- Explore [Error Handling](error-handling.md) for robust workflow design  
- See [Triggers and Scheduling](triggers-scheduling.md) for automated execution