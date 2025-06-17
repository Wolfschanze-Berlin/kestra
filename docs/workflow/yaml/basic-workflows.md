# Basic Workflows

This guide covers the fundamentals of creating YAML workflows in Kestra, from the simplest examples to common patterns you'll use in everyday workflow development.

## Workflow Structure

Every Kestra workflow must have three required properties:

1. `id` - Unique identifier for the workflow
2. `namespace` - Organizational grouping for workflows  
3. `tasks` - List of tasks to execute

### Minimal Workflow

The simplest possible workflow:

```yaml
id: minimal
namespace: tutorial
tasks:
  - id: log-message
    type: io.kestra.plugin.core.log.Log
    message: "This is a minimal workflow"
```

### Complete Basic Workflow

A more comprehensive example showing common properties:

```yaml
id: basic-example
namespace: tutorial
description: |
  A basic workflow demonstrating common properties and features.
  This workflow shows how to use variables, inputs, and basic tasks.

labels:
  team: data-engineering
  environment: development
  project: tutorial

inputs:
  - id: user_name
    type: STRING
    required: false
    defaults: "Guest"
    description: "Name of the user to greet"

variables:
  greeting_prefix: "Hello"
  timestamp_format: "yyyy-MM-dd HH:mm:ss"

tasks:
  - id: welcome
    type: io.kestra.plugin.core.log.Log
    message: "{{ vars.greeting_prefix }}, {{ inputs.user_name }}!"
    
  - id: show-info
    type: io.kestra.plugin.core.log.Log
    message: |
      Workflow Information:
      - Flow ID: {{ flow.id }}
      - Namespace: {{ flow.namespace }}
      - Execution ID: {{ execution.id }}
      - Started at: {{ execution.startDate | date(vars.timestamp_format) }}
      
  - id: completion
    type: io.kestra.plugin.core.debug.Return
    format: "Workflow completed successfully at {{ now() | date(vars.timestamp_format) }}"

outputs:
  - id: completion_message
    type: STRING
    value: "{{ outputs.completion.value }}"
```

## Core Task Properties

All tasks share these common properties:

### Required Properties

- `id`: Unique identifier within the workflow
- `type`: Full Java class name of the task type

### Optional Properties

- `description`: Human-readable description of what the task does
- `disabled`: Set to `true` to skip the task during execution
- `timeout`: Maximum time allowed for task completion (e.g., `PT30M` for 30 minutes)
- `retry`: Retry configuration for handling task failures
- `allowFailure`: Continue workflow execution even if this task fails

### Example with All Properties

```yaml
id: comprehensive-task-example
namespace: tutorial
tasks:
  - id: robust-task
    type: io.kestra.plugin.core.log.Log
    description: "A task demonstrating all common properties"
    message: "Processing important data..."
    timeout: PT10M
    allowFailure: false
    retry:
      maxAttempt: 3
      type: exponential
      interval: PT30S
      maxDuration: PT5M
```

## Working with Inputs

Inputs make workflows dynamic and reusable by accepting values at runtime.

### Input Types

```yaml
id: input-types-example
namespace: tutorial
description: "Demonstrates different input types"

inputs:
  - id: text_input
    type: STRING
    required: true
    description: "A required text input"
    
  - id: number_input
    type: INT
    defaults: 42
    description: "An optional number with default value"
    
  - id: flag_input
    type: BOOL
    defaults: false
    description: "A boolean flag"
    
  - id: date_input
    type: DATETIME
    required: false
    description: "An optional date/time input"
    
  - id: file_input
    type: FILE
    required: false
    description: "File upload input"
    
  - id: choice_input
    type: SELECT
    values:
      - development
      - staging
      - production
    defaults: development
    description: "Environment selection"

tasks:
  - id: display-inputs
    type: io.kestra.plugin.core.log.Log
    message: |
      Inputs received:
      - Text: {{ inputs.text_input }}
      - Number: {{ inputs.number_input }}
      - Flag: {{ inputs.flag_input }}
      - Date: {{ inputs.date_input | default("Not provided") }}
      - Environment: {{ inputs.choice_input }}
```

### Input Validation and Dependencies

```yaml
id: input-validation-example
namespace: tutorial

inputs:
  - id: enable_processing
    type: BOOL
    defaults: false
    description: "Enable data processing"
    
  - id: processing_type
    type: SELECT
    values:
      - batch
      - streaming
      - real-time
    dependsOn:
      inputs:
        - enable_processing
      condition: "{{ inputs.enable_processing }}"
    description: "Type of processing (only shown when processing is enabled)"
    
  - id: batch_size
    type: INT
    defaults: 1000
    dependsOn:
      inputs:
        - processing_type
      condition: "{{ inputs.processing_type == 'batch' }}"
    description: "Batch size (only for batch processing)"

tasks:
  - id: conditional-processing
    type: io.kestra.plugin.core.log.Log
    message: "Processing {{ inputs.processing_type }} with batch size {{ inputs.batch_size | default('N/A') }}"
    runIf: "{{ inputs.enable_processing }}"
```

## Working with Variables

Variables allow you to define reusable values and computed expressions.

### Simple Variables

```yaml
id: variables-example
namespace: tutorial

variables:
  app_name: "Data Pipeline"
  version: "1.2.3"
  environment: "production"
  max_retries: 5
  timeout_minutes: 30

tasks:
  - id: app-info
    type: io.kestra.plugin.core.log.Log
    message: "Starting {{ vars.app_name }} v{{ vars.version }} in {{ vars.environment }}"
    
  - id: config-info
    type: io.kestra.plugin.core.log.Log
    message: "Configuration: max_retries={{ vars.max_retries }}, timeout={{ vars.timeout_minutes }}min"
```

### Computed Variables

```yaml
id: computed-variables-example
namespace: tutorial

variables:
  base_url: "https://api.example.com"
  api_version: "v2"
  full_api_url: "{{ vars.base_url }}/{{ vars.api_version }}"
  current_date: "{{ now() | date('yyyy-MM-dd') }}"
  is_weekend: "{{ now() | date('u') as d }}{{ d >= 6 }}"

tasks:
  - id: show-computed
    type: io.kestra.plugin.core.log.Log
    message: |
      Computed values:
      - API URL: {{ vars.full_api_url }}
      - Date: {{ vars.current_date }}
      - Is weekend: {{ render(vars.is_weekend) }}
```

## Task Outputs and Data Flow

Tasks can produce outputs that subsequent tasks can use.

### Basic Outputs

```yaml
id: task-outputs-example
namespace: tutorial

tasks:
  - id: generate-data
    type: io.kestra.plugin.core.debug.Return
    format: |
      {
        "timestamp": "{{ now() }}",
        "user_count": 150,
        "status": "active"
      }
      
  - id: process-data
    type: io.kestra.plugin.core.log.Log
    message: |
      Processing data from previous task:
      Raw output: {{ outputs.generate-data.value }}
      User count: {{ fromJson(outputs.generate-data.value).user_count }}
      Status: {{ fromJson(outputs.generate-data.value).status }}
```

### Flow Outputs

Workflows can also produce outputs for use by other workflows:

```yaml
id: workflow-outputs-example
namespace: tutorial

tasks:
  - id: calculate
    type: io.kestra.plugin.core.debug.Return
    format: "{{ 10 * 5 }}"
    
  - id: summary
    type: io.kestra.plugin.core.debug.Return
    format: |
      {
        "calculation_result": {{ outputs.calculate.value }},
        "processed_at": "{{ now() }}",
        "workflow_id": "{{ flow.id }}"
      }

outputs:
  - id: result
    type: STRING
    value: "{{ outputs.calculate.value }}"
    
  - id: summary_json
    type: JSON
    value: "{{ outputs.summary.value }}"
    
  - id: execution_info
    type: STRING
    value: "Workflow {{ flow.id }} completed at {{ now() }}"
```

## Common Patterns

### Data Validation Pattern

```yaml
id: validation-pattern
namespace: tutorial

inputs:
  - id: data_file
    type: FILE
    required: true

tasks:
  - id: validate-file
    type: io.kestra.plugin.core.log.Log
    message: "Validating uploaded file: {{ inputs.data_file }}"
    
  - id: check-size
    type: io.kestra.plugin.core.log.Log
    message: "File size: {{ fileSize(inputs.data_file) }} bytes"
    
  - id: process-file
    type: io.kestra.plugin.core.log.Log
    message: "Processing file..."
    runIf: "{{ fileSize(inputs.data_file) > 0 }}"
```

### Configuration Pattern

```yaml
id: configuration-pattern
namespace: tutorial

variables:
  # Environment-specific configuration
  database:
    host: "localhost"
    port: 5432
    database: "analytics"
  
  # Feature flags
  features:
    enable_caching: true
    enable_notifications: false
    debug_mode: false

tasks:
  - id: connect-database
    type: io.kestra.plugin.core.log.Log
    message: "Connecting to {{ vars.database.host }}:{{ vars.database.port }}/{{ vars.database.database }}"
    
  - id: cache-check
    type: io.kestra.plugin.core.log.Log
    message: "Caching is {{ vars.features.enable_caching ? 'enabled' : 'disabled' }}"
    
  - id: debug-info
    type: io.kestra.plugin.core.log.Log
    message: "Debug information: {{ printContext() }}"
    runIf: "{{ vars.features.debug_mode }}"
```

## Next Steps

- Learn about [Flow Control](flow-control.md) for sequential, parallel, and conditional execution
- Explore [Variables and Data](variables-and-data.md) for advanced data handling
- See [Error Handling](error-handling.md) for robust workflow design