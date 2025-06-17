# Variables and Data

This guide covers how to work with variables, inputs, outputs, and data flow in Kestra workflows, including the powerful Pebble templating engine.

## Variables

Variables allow you to define reusable values throughout your workflow, making it easier to maintain and configure.

### Basic Variables

```yaml
id: basic-variables-example
namespace: tutorial

variables:
  app_name: "Data Processing Pipeline"
  version: "2.1.0"
  max_retries: 3
  timeout_seconds: 300
  debug_mode: false
  
tasks:
  - id: show-config
    type: io.kestra.plugin.core.log.Log
    message: |
      Application: {{ vars.app_name }} v{{ vars.version }}
      Max retries: {{ vars.max_retries }}
      Timeout: {{ vars.timeout_seconds }}s
      Debug: {{ vars.debug_mode }}
```

### Structured Variables

```yaml
id: structured-variables-example
namespace: tutorial

variables:
  database:
    host: "db.example.com"
    port: 5432
    name: "analytics"
    ssl_enabled: true
    
  api:
    base_url: "https://api.example.com"
    version: "v2"
    timeout: 30
    
  environment:
    name: "production"
    region: "us-east-1"
    monitoring_enabled: true

tasks:
  - id: database-connection
    type: io.kestra.plugin.core.log.Log
    message: "Connecting to {{ vars.database.host }}:{{ vars.database.port }}/{{ vars.database.name }}"
    
  - id: api-setup
    type: io.kestra.plugin.core.log.Log
    message: "API endpoint: {{ vars.api.base_url }}/{{ vars.api.version }}"
    
  - id: environment-info
    type: io.kestra.plugin.core.log.Log
    message: "Running in {{ vars.environment.name }} ({{ vars.environment.region }})"
```

### Computed Variables

Variables can contain Pebble expressions that are evaluated at runtime:

```yaml
id: computed-variables-example
namespace: tutorial

variables:
  # Basic computed values
  current_date: "{{ now() | date('yyyy-MM-dd') }}"
  current_timestamp: "{{ now() | timestamp }}"
  
  # URL construction
  base_url: "https://api.example.com"
  api_version: "v2"
  endpoint: "{{ vars.base_url }}/{{ vars.api_version }}"
  
  # Conditional values
  environment: "production"
  log_level: "{{ vars.environment == 'production' ? 'INFO' : 'DEBUG' }}"
  
  # Mathematical calculations
  batch_size: 1000
  total_batches: 10
  total_records: "{{ vars.batch_size * vars.total_batches }}"
  
  # Date calculations
  cutoff_date: "{{ now() | dateAdd(-7, 'DAYS') | date('yyyy-MM-dd') }}"

tasks:
  - id: show-computed
    type: io.kestra.plugin.core.log.Log
    message: |
      Computed Variables:
      - Date: {{ vars.current_date }}
      - Timestamp: {{ vars.current_timestamp }}
      - API Endpoint: {{ vars.endpoint }}
      - Log Level: {{ render(vars.log_level) }}
      - Total Records: {{ render(vars.total_records) }}
      - Cutoff Date: {{ render(vars.cutoff_date) }}
```

## Inputs

Inputs make workflows dynamic and reusable by accepting values at execution time.

### Input Types and Validation

```yaml
id: input-types-comprehensive
namespace: tutorial

inputs:
  # String inputs
  - id: user_name
    type: STRING
    required: true
    description: "Name of the user"
    
  - id: description
    type: STRING
    required: false
    defaults: "No description provided"
    
  # Numeric inputs
  - id: batch_size
    type: INT
    required: false
    defaults: 1000
    description: "Number of records to process at once"
    
  - id: timeout_minutes
    type: FLOAT
    required: false
    defaults: 5.5
    description: "Timeout in minutes"
    
  # Boolean inputs
  - id: enable_notifications
    type: BOOL
    defaults: false
    description: "Send notifications on completion"
    
  # Date/time inputs
  - id: start_date
    type: DATE
    required: false
    description: "Processing start date (YYYY-MM-DD)"
    
  - id: execution_time
    type: DATETIME
    required: false
    description: "Scheduled execution time"
    
  # Selection inputs
  - id: environment
    type: SELECT
    values:
      - development
      - staging
      - production
    defaults: development
    description: "Target environment"
    
  - id: data_sources
    type: MULTISELECT
    values:
      - database
      - api
      - files
      - streaming
    description: "Select data sources to process"
    
  # File inputs
  - id: config_file
    type: FILE
    required: false
    description: "Configuration file upload"
    
  # JSON inputs
  - id: custom_config
    type: JSON
    required: false
    defaults: |
      {
        "processing_mode": "batch",
        "parallel_workers": 4
      }
    description: "Custom configuration as JSON"

tasks:
  - id: display-inputs
    type: io.kestra.plugin.core.log.Log
    message: |
      Inputs received:
      - User: {{ inputs.user_name }}
      - Description: {{ inputs.description }}
      - Batch Size: {{ inputs.batch_size }}
      - Timeout: {{ inputs.timeout_minutes }} minutes
      - Notifications: {{ inputs.enable_notifications }}
      - Start Date: {{ inputs.start_date | default("Not specified") }}
      - Environment: {{ inputs.environment }}
      - Data Sources: {{ inputs.data_sources | default([]) | join(", ") }}
      - Custom Config: {{ inputs.custom_config | toJson }}
```

### Conditional Inputs

Use `dependsOn` to show inputs based on other input values:

```yaml
id: conditional-inputs-example
namespace: tutorial

inputs:
  - id: processing_mode
    type: SELECT
    values:
      - batch
      - streaming
      - realtime
    defaults: batch
    description: "Data processing mode"
    
  - id: batch_size
    type: INT
    defaults: 1000
    dependsOn:
      inputs:
        - processing_mode
      condition: "{{ inputs.processing_mode == 'batch' }}"
    description: "Batch size (only for batch processing)"
    
  - id: stream_buffer_size
    type: INT
    defaults: 100
    dependsOn:
      inputs:
        - processing_mode
      condition: "{{ inputs.processing_mode == 'streaming' }}"
    description: "Stream buffer size (only for streaming)"
    
  - id: enable_advanced_options
    type: BOOL
    defaults: false
    description: "Enable advanced configuration options"
    
  - id: advanced_config
    type: JSON
    dependsOn:
      inputs:
        - enable_advanced_options
      condition: "{{ inputs.enable_advanced_options }}"
    defaults: |
      {
        "retry_policy": "exponential",
        "max_memory": "2GB",
        "compression": true
      }
    description: "Advanced configuration (shown only when enabled)"

tasks:
  - id: process-with-config
    type: io.kestra.plugin.core.log.Log
    message: |
      Processing Configuration:
      - Mode: {{ inputs.processing_mode }}
      {% if inputs.processing_mode == 'batch' -%}
      - Batch Size: {{ inputs.batch_size }}
      {% elif inputs.processing_mode == 'streaming' -%}
      - Buffer Size: {{ inputs.stream_buffer_size }}
      {% endif -%}
      {% if inputs.enable_advanced_options -%}
      - Advanced Config: {{ inputs.advanced_config | toJson }}
      {% endif %}
```

## Outputs

Outputs allow tasks and workflows to produce data that can be consumed by other tasks or external systems.

### Task Outputs

```yaml
id: task-outputs-example
namespace: tutorial

tasks:
  - id: data-generator
    type: io.kestra.plugin.core.debug.Return
    format: |
      {
        "generated_at": "{{ now() }}",
        "record_count": 1500,
        "data_quality_score": 0.95,
        "processing_stats": {
          "duration_ms": 2340,
          "memory_used": "128MB"
        }
      }
      
  - id: data-processor
    type: io.kestra.plugin.core.debug.Return
    format: |
      {
        "input_records": {{ fromJson(outputs.data-generator.value).record_count }},
        "processed_records": {{ fromJson(outputs.data-generator.value).record_count * 0.98 | round }},
        "quality_score": {{ fromJson(outputs.data-generator.value).data_quality_score }},
        "processed_at": "{{ now() }}"
      }
      
  - id: summary-report
    type: io.kestra.plugin.core.log.Log
    message: |
      Processing Summary:
      - Generated: {{ fromJson(outputs.data-generator.value).record_count }} records
      - Processed: {{ fromJson(outputs.data-processor.value).processed_records }} records
      - Quality Score: {{ fromJson(outputs.data-processor.value).quality_score }}
      - Success Rate: {{ (fromJson(outputs.data-processor.value).processed_records / fromJson(outputs.data-generator.value).record_count * 100) | round(2) }}%
```

### Flow Outputs

Workflows can produce outputs for use by other workflows or external systems:

```yaml
id: workflow-outputs-example
namespace: tutorial

inputs:
  - id: dataset_name
    type: STRING
    required: true

tasks:
  - id: analyze-data
    type: io.kestra.plugin.core.debug.Return
    format: |
      {
        "dataset": "{{ inputs.dataset_name }}",
        "analysis": {
          "total_rows": 50000,
          "columns": 12,
          "null_percentage": 2.3,
          "data_types": ["string", "integer", "float", "datetime"]
        },
        "recommendations": [
          "Consider indexing on timestamp column",
          "Validate email format in contact_email field"
        ]
      }
      
  - id: generate-metadata
    type: io.kestra.plugin.core.debug.Return
    format: |
      {
        "dataset_name": "{{ inputs.dataset_name }}",
        "analyzed_at": "{{ now() }}",
        "workflow_id": "{{ flow.id }}",
        "execution_id": "{{ execution.id }}",
        "version": "1.0"
      }

outputs:
  - id: analysis_results
    type: JSON
    value: "{{ outputs.analyze-data.value }}"
    description: "Complete data analysis results"
    
  - id: row_count
    type: INT
    value: "{{ fromJson(outputs.analyze-data.value).analysis.total_rows }}"
    description: "Total number of rows in the dataset"
    
  - id: data_quality_score
    type: FLOAT
    value: "{{ 100 - fromJson(outputs.analyze-data.value).analysis.null_percentage }}"
    description: "Data quality score as percentage"
    
  - id: metadata
    type: JSON
    value: "{{ outputs.generate-metadata.value }}"
    description: "Processing metadata and timestamps"
    
  - id: summary
    type: STRING
    value: "Analysis of {{ inputs.dataset_name }} completed: {{ fromJson(outputs.analyze-data.value).analysis.total_rows }} rows, {{ fromJson(outputs.analyze-data.value).analysis.columns }} columns"
    description: "Human-readable summary"
```

## Pebble Templating

Kestra uses the Pebble templating engine for dynamic content generation.

### Basic Expressions

```yaml
id: pebble-basics-example
namespace: tutorial

variables:
  user_name: "Alice"
  items: [1, 2, 3, 4, 5]

tasks:
  - id: basic-expressions
    type: io.kestra.plugin.core.log.Log
    message: |
      Basic Expressions:
      - Flow ID: {{ flow.id }}
      - Namespace: {{ flow.namespace }}
      - Execution ID: {{ execution.id }}
      - Current time: {{ now() }}
      - User: {{ vars.user_name }}
      - Item count: {{ vars.items | length }}
      - First item: {{ vars.items | first }}
      - Last item: {{ vars.items | last }}
```

### String Manipulation

```yaml
id: string-manipulation-example
namespace: tutorial

variables:
  message: "  Hello, World!  "
  email: "user@example.com"
  data: "apple,banana,cherry"

tasks:
  - id: string-operations
    type: io.kestra.plugin.core.log.Log
    message: |
      String Operations:
      - Original: "{{ vars.message }}"
      - Trimmed: "{{ vars.message | trim }}"
      - Uppercase: "{{ vars.message | trim | upper }}"
      - Lowercase: "{{ vars.message | trim | lower }}"
      - Capitalized: "{{ vars.message | trim | capitalize }}"
      - Email domain: "{{ vars.email | split('@') | last }}"
      - Data items: {{ vars.data | split(',') | toJson }}
      - Joined: "{{ vars.data | split(',') | join(' | ') }}"
```

### Date and Time Operations

```yaml
id: datetime-operations-example
namespace: tutorial

tasks:
  - id: datetime-examples
    type: io.kestra.plugin.core.log.Log
    message: |
      Date/Time Operations:
      - Now: {{ now() }}
      - Formatted: {{ now() | date('yyyy-MM-dd HH:mm:ss') }}
      - ISO format: {{ now() | date('yyyy-MM-dd\'T\'HH:mm:ss\'Z\'') }}
      - Date only: {{ now() | date('yyyy-MM-dd') }}
      - Time only: {{ now() | date('HH:mm:ss') }}
      - Unix timestamp: {{ now() | timestamp }}
      - Add 1 day: {{ now() | dateAdd(1, 'DAYS') | date('yyyy-MM-dd') }}
      - Subtract 1 week: {{ now() | dateAdd(-7, 'DAYS') | date('yyyy-MM-dd') }}
      - Start of month: {{ now() | date('yyyy-MM-01') }}
```

### Mathematical Operations

```yaml
id: math-operations-example
namespace: tutorial

variables:
  numbers: [10, 25, 5, 40, 15]
  price: 99.99
  quantity: 3

tasks:
  - id: math-examples
    type: io.kestra.plugin.core.log.Log
    message: |
      Mathematical Operations:
      - Sum: {{ 10 + 20 + 30 }}
      - Product: {{ vars.price * vars.quantity }}
      - Average: {{ (vars.numbers | sum) / (vars.numbers | length) }}
      - Max value: {{ max(vars.numbers) }}
      - Min value: {{ min(vars.numbers) }}
      - Rounded: {{ vars.price | round(0) }}
      - Absolute: {{ abs(-42) }}
      - Random 0-100: {{ random() * 100 | round(0) }}
```

### JSON and Data Manipulation

```yaml
id: json-manipulation-example
namespace: tutorial

variables:
  json_data: |
    {
      "users": [
        {"id": 1, "name": "Alice", "age": 30},
        {"id": 2, "name": "Bob", "age": 25},
        {"id": 3, "name": "Charlie", "age": 35}
      ],
      "metadata": {
        "total": 3,
        "created_at": "2024-01-01"
      }
    }

tasks:
  - id: json-operations
    type: io.kestra.plugin.core.log.Log
    message: |
      JSON Operations:
      - User count: {{ fromJson(vars.json_data).users | length }}
      - First user: {{ fromJson(vars.json_data).users | first | toJson }}
      - User names: {{ fromJson(vars.json_data).users | map(attribute='name') | join(', ') }}
      - Average age: {{ (fromJson(vars.json_data).users | map(attribute='age') | sum) / (fromJson(vars.json_data).users | length) }}
      - Metadata: {{ fromJson(vars.json_data).metadata | toJson }}
      
  - id: filter-users
    type: io.kestra.plugin.core.debug.Return
    format: |
      {
        "adults": {{ fromJson(vars.json_data).users | selectattr('age', 'ge', 30) | list | toJson }},
        "names_starting_with_a": {{ fromJson(vars.json_data).users | selectattr('name', 'match', '^A.*') | map(attribute='name') | list | toJson }}
      }
```

### Conditional Logic in Templates

```yaml
id: conditional-templates-example
namespace: tutorial

inputs:
  - id: environment
    type: SELECT
    values: [dev, staging, prod]
    defaults: dev
    
  - id: user_count
    type: INT
    defaults: 150

tasks:
  - id: conditional-logic
    type: io.kestra.plugin.core.log.Log
    message: |
      Conditional Logic:
      {% if inputs.environment == 'prod' %}
      🚨 PRODUCTION ENVIRONMENT - Use caution!
      {% elif inputs.environment == 'staging' %}
      🔧 Staging environment - Safe for testing
      {% else %}
      🛠️ Development environment - Free to experiment
      {% endif %}
      
      User Count Analysis:
      {% if inputs.user_count > 1000 %}
      Large user base - consider performance optimizations
      {% elif inputs.user_count > 100 %}
      Medium user base - standard processing
      {% else %}
      Small user base - simple processing sufficient
      {% endif %}
      
      Environment Config:
      - Database: {{ inputs.environment == 'prod' ? 'prod-db.example.com' : 'test-db.example.com' }}
      - Cache TTL: {{ inputs.environment == 'prod' ? '3600' : '300' }} seconds
      - Debug Mode: {{ inputs.environment != 'prod' ? 'enabled' : 'disabled' }}
```

### Loops in Templates

```yaml
id: template-loops-example
namespace: tutorial

variables:
  services: 
    - name: "web-server"
      port: 8080
      health_check: "/health"
    - name: "api-server"
      port: 3000
      health_check: "/api/health"
    - name: "database"
      port: 5432
      health_check: null

tasks:
  - id: service-status
    type: io.kestra.plugin.core.log.Log
    message: |
      Service Status Check:
      {% for service in vars.services %}
      {{ loop.index }}. {{ service.name }}:
         - Port: {{ service.port }}
         - Health Check: {{ service.health_check | default('Not configured') }}
         {% if loop.first %}(Primary Service){% endif %}
         {% if loop.last %}(Last Service){% endif %}
      {% endfor %}
      
      Summary: {{ vars.services | length }} services configured
```

## Data Transformation Patterns

### ETL Pattern with Data Validation

```yaml
id: etl-data-validation-pattern
namespace: tutorial

inputs:
  - id: source_data
    type: JSON
    defaults: |
      [
        {"id": 1, "name": "John Doe", "email": "john@example.com", "age": 30},
        {"id": 2, "name": "Jane Smith", "email": "jane@example.com", "age": 25},
        {"id": 3, "name": "Bob Wilson", "email": "invalid-email", "age": -5}
      ]

tasks:
  # Extract
  - id: extract-data
    type: io.kestra.plugin.core.debug.Return
    format: "{{ inputs.source_data | toJson }}"
    
  # Validate
  - id: validate-data
    type: io.kestra.plugin.core.debug.Return
    format: |
      {
        "valid_records": {{ inputs.source_data | selectattr('age', 'ge', 0) | selectattr('email', 'match', '.*@.*\\..*') | list | toJson }},
        "invalid_records": {{ inputs.source_data | rejectattr('age', 'ge', 0) | list | toJson }},
        "email_errors": {{ inputs.source_data | rejectattr('email', 'match', '.*@.*\\..*') | list | toJson }}
      }
      
  # Transform
  - id: transform-data
    type: io.kestra.plugin.core.debug.Return
    format: |
      {
        "transformed_records": [
          {% for record in fromJson(outputs.validate-data.value).valid_records %}
          {
            "id": {{ record.id }},
            "full_name": "{{ record.name | upper }}",
            "email_domain": "{{ record.email | split('@') | last }}",
            "age_group": "{% if record.age < 25 %}young{% elif record.age < 40 %}adult{% else %}senior{% endif %}",
            "processed_at": "{{ now() }}"
          }{% if not loop.last %},{% endif %}
          {% endfor %}
        ],
        "validation_summary": {
          "total_input": {{ inputs.source_data | length }},
          "valid_records": {{ fromJson(outputs.validate-data.value).valid_records | length }},
          "invalid_records": {{ fromJson(outputs.validate-data.value).invalid_records | length }}
        }
      }
      
  # Load (Log results)
  - id: load-results
    type: io.kestra.plugin.core.log.Log
    message: |
      ETL Process Complete:
      - Input Records: {{ fromJson(outputs.transform-data.value).validation_summary.total_input }}
      - Valid Records: {{ fromJson(outputs.transform-data.value).validation_summary.valid_records }}
      - Invalid Records: {{ fromJson(outputs.transform-data.value).validation_summary.invalid_records }}
      - Success Rate: {{ (fromJson(outputs.transform-data.value).validation_summary.valid_records / fromJson(outputs.transform-data.value).validation_summary.total_input * 100) | round(1) }}%

outputs:
  - id: processed_data
    type: JSON
    value: "{{ outputs.transform-data.value }}"
    
  - id: success_rate
    type: FLOAT
    value: "{{ (fromJson(outputs.transform-data.value).validation_summary.valid_records / fromJson(outputs.transform-data.value).validation_summary.total_input * 100) | round(1) }}"
```

## Best Practices

### 1. Use Meaningful Variable Names
```yaml
# Good
variables:
  database_connection_timeout: 30
  api_retry_attempts: 3
  batch_processing_size: 1000

# Avoid
variables:
  timeout: 30
  retries: 3
  size: 1000
```

### 2. Validate Input Data
```yaml
inputs:
  - id: email
    type: STRING
    required: true
    description: "Valid email address"

tasks:
  - id: validate-email
    type: io.kestra.plugin.core.log.Log
    message: "Processing email: {{ inputs.email }}"
    runIf: "{{ inputs.email | match('.*@.*\\..*') }}"
```

### 3. Use Default Values Appropriately
```yaml
variables:
  # Provide sensible defaults
  batch_size: "{{ inputs.batch_size | default(1000) }}"
  timeout: "{{ inputs.timeout | default('PT30M') }}"
  debug_mode: "{{ inputs.debug_mode | default(false) }}"
```

### 4. Structure Complex Data
```yaml
variables:
  config:
    database:
      host: "{{ inputs.db_host | default('localhost') }}"
      port: "{{ inputs.db_port | default(5432) }}"
    processing:
      batch_size: "{{ inputs.batch_size | default(1000) }}"
      parallel_workers: "{{ inputs.workers | default(4) }}"
```

## Next Steps

- Learn about [Error Handling](error-handling.md) for robust workflow design
- Explore [Triggers and Scheduling](triggers-scheduling.md) for automated execution  
- See [Best Practices](best-practices.md) for maintainable workflows