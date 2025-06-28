# GitHub Copilot Custom Instructions for Kestra

You are assisting with Kestra, an open-source declarative orchestration and scheduling platform that helps developers and teams create, manage, and execute complex workflows using YAML configuration.

## Repository Context

This is the Kestra core repository containing the workflow orchestration platform with extensive plugin ecosystem support. The project includes comprehensive documentation for LLM-assisted workflow development located in the `docs/` directory.

## Key Documentation References

### Core Workflow Documentation
- `docs/llm_txt/workflow-basics.txt` - Fundamental workflow structure and anatomy
- `docs/llm_txt/inputs-and-expressions.txt` - Input types and Pebble expression language
- `docs/llm_txt/flow-control-tasks.txt` - Sequential, parallel, and conditional execution
- `docs/llm_txt/utility-tasks.txt` - Debug, logging, state, and system management
- `docs/llm_txt/triggers-and-listeners.txt` - Event-driven execution patterns
- `docs/llm_txt/best-practices.txt` - Production guidelines and optimization

### Plugin Ecosystem (700+ plugins)
- `docs/llm_txt/plugins-directory.txt` - Complete plugins directory and quick reference
- `docs/llm_txt/plugin-ecosystem.txt` - Detailed plugin examples and integration patterns
- `docs/llm_txt/plugin-categories.txt` - Categorized plugin reference by use case

### Sample Workflows
- `docs/workflows/` - Production-ready workflow examples with mermaid diagrams
- `docs/component-dependencies.md` - System architecture and component relationships

## Workflow Development Guidelines

### When writing Kestra YAML workflows:

1. **Structure**: Always follow the standard workflow anatomy:
   ```yaml
   id: workflow-name
   namespace: organization.team
   description: "Clear description of workflow purpose"
   
   inputs:
     - id: parameter_name
       type: STRING  # STRING, INT, FLOAT, BOOLEAN, JSON, DATETIME, FILE, SECRET
       description: "Parameter description"
   
   labels:
     environment: "production"
     team: "team-name"
   
   tasks:
     - id: task-name
       type: plugin.task.type
       # task configuration
   
   triggers:
     - id: trigger-name
       type: trigger.type
       # trigger configuration
   
   errors:
     - id: error-handler
       type: error.handler.type
       # error handling
   ```

2. **Task Types**: Reference the extensive plugin ecosystem:
   - **Core Flow Control**: `Sequential`, `Parallel`, `ForEach`, `If`, `Switch`, `Subflow`
   - **Utility Tasks**: `Debug.Return`, `Log`, `Sleep`, `Pause`, `State.Set/Get`
   - **Cloud Providers**: AWS (S3, Lambda, EC2), GCP (BigQuery, Cloud Storage), Azure
   - **Databases**: JDBC, MongoDB, Elasticsearch, Redis, PostgreSQL, MySQL
   - **Data Processing**: DBT, Spark, Databricks, Airbyte
   - **AI/ML**: OpenAI, LangChain4J, Weaviate
   - **DevOps**: Kubernetes, Docker, Terraform, Git, Ansible
   - **Messaging**: Kafka, Pulsar, NATS, AMQP, MQTT

3. **Expression Language**: Use Pebble templating for dynamic behavior:
   ```yaml
   # Input access
   format: "Processing {{inputs.parameter_name}}"
   
   # Task output access
   condition: "{{outputs.previous_task.status == 'SUCCESS'}}"
   
   # Conditional logic
   condition: "{{inputs.environment == 'production'}}"
   
   # Date operations
   date: "{{now() | date('yyyy-MM-dd')}}"
   
   # JSON processing
   host: "{{inputs.config | json_path('$.database.host')}}"
   
   # Default values
   value: "{{inputs.optional_param ?? 'default_value'}}"
   ```

4. **Error Handling**: Always include appropriate error handling:
   ```yaml
   errors:
     - id: failure-notification
       type: io.kestra.plugin.core.log.Log
       message: "Workflow failed: {{error.message}}"
       level: ERROR
   ```

5. **Best Practices**:
   - Use descriptive task IDs and workflow names
   - Include proper input validation and descriptions
   - Implement retry logic for external operations
   - Add appropriate logging and monitoring
   - Use secrets for sensitive data
   - Apply proper resource management
   - Follow naming conventions: kebab-case for IDs, descriptive namespaces

### Code Quality Standards

- **YAML Syntax**: Ensure proper indentation (2 spaces) and valid YAML structure
- **Task Configuration**: Use correct plugin task types and required parameters
- **Expression Syntax**: Validate Pebble expressions for dynamic content
- **Resource Management**: Consider memory, CPU, and storage requirements
- **Security**: Use proper secret handling and access controls

### Common Patterns

1. **ETL Pipeline**:
   ```yaml
   tasks:
     - id: extract
       type: io.kestra.plugin.jdbc.postgresql.Query
       
     - id: transform
       type: io.kestra.plugin.core.flow.Sequential
       
     - id: load
       type: io.kestra.plugin.aws.s3.Upload
   ```

2. **API Integration**:
   ```yaml
   tasks:
     - id: api-call
       type: io.kestra.plugin.core.http.Request
       retry:
         maxAttempt: 3
         interval: PT30S
   ```

3. **Conditional Execution**:
   ```yaml
   tasks:
     - id: check-environment
       type: io.kestra.plugin.core.flow.If
       condition: "{{inputs.environment == 'production'}}"
       then:
         - id: production-tasks
           type: io.kestra.plugin.core.flow.Sequential
   ```

## Development Context

- **Language**: YAML workflow definitions, Java core platform
- **Testing**: Include proper workflow testing and validation
- **Documentation**: Reference comprehensive docs in `docs/llm_txt/` for accurate implementations
- **Plugin Ecosystem**: Leverage 700+ available plugins across 13 categories
- **Architecture**: Cloud-native, scalable orchestration platform

## Helpful Resources

When suggesting workflows or troubleshooting:
1. Check plugin documentation in `docs/llm_txt/plugin-ecosystem.txt`
2. Reference workflow examples in `docs/workflows/`
3. Follow best practices from `docs/llm_txt/best-practices.txt`
4. Use expression patterns from `docs/llm_txt/inputs-and-expressions.txt`

Always prioritize production-ready, maintainable, and well-documented workflow solutions.