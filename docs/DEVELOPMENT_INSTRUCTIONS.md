# Kestra Workflow Development Instructions

This document provides custom instructions for AI assistants and developers working with Kestra workflows, based on the comprehensive documentation in this repository.

## Quick Start Guide

### Using the Documentation

The `docs/` folder contains comprehensive Kestra workflow documentation organized for optimal AI assistant understanding:

#### 📖 Core Documentation (`docs/llm_txt/`)
- **workflow-basics.txt** - Fundamental concepts and workflow anatomy
- **inputs-and-expressions.txt** - Input types and Pebble expression language
- **flow-control-tasks.txt** - Sequential, parallel, and conditional execution
- **utility-tasks.txt** - Debug, logging, state, and system management
- **triggers-and-listeners.txt** - Event-driven execution patterns
- **best-practices.txt** - Production guidelines and optimization

#### 🔌 Plugin Ecosystem (`docs/llm_txt/`)
- **plugins-directory.txt** - Complete 700+ plugins directory
- **plugin-ecosystem.txt** - Detailed plugin examples and integration patterns
- **plugin-categories.txt** - Categorized plugin reference by use case
- **advanced-workflow-examples.txt** - Production-ready workflow examples

#### 📊 Sample Workflows (`docs/workflows/`)
- **data-processing/** - ETL pipelines with validation and error handling
- **api-integration/** - API integration with polling and circuit breaker patterns
- **deployment/** - Multi-environment deployment with approval gates
- **monitoring/** - System monitoring with automated alerting

## AI Assistant Instructions

### Context Setup
When working with Kestra workflows, always reference the documentation in `docs/llm_txt/` for accurate plugin usage, task types, and best practices. The documentation covers:

- **700+ plugins** across 13 categories (Cloud, Database, AI/ML, DevOps, etc.)
- **Complete workflow patterns** for common use cases
- **Production best practices** for security, performance, and maintainability
- **Expression language** examples with Pebble templating

### Workflow Generation Guidelines

1. **Use Standard Structure**: Always follow the documented workflow anatomy from `workflow-basics.txt`
2. **Leverage Plugin Ecosystem**: Reference `plugin-ecosystem.txt` for appropriate task types
3. **Apply Best Practices**: Follow guidelines from `best-practices.txt` for production-ready workflows
4. **Include Error Handling**: Implement proper retry logic and error recovery patterns
5. **Use Dynamic Expressions**: Leverage Pebble templating patterns from `inputs-and-expressions.txt`

### Example Workflow Template

```yaml
id: workflow-name
namespace: organization.team
description: "Clear description of workflow purpose"

inputs:
  - id: parameter_name
    type: STRING  # STRING, INT, FLOAT, BOOLEAN, JSON, DATETIME, FILE, SECRET
    description: "Parameter description"
    defaults: "default_value"

labels:
  environment: "production"
  team: "team-name"
  version: "1.0.0"

tasks:
  - id: main-task
    type: io.kestra.plugin.core.debug.Return
    format: "Processing {{inputs.parameter_name}}"

triggers:
  - id: schedule
    type: io.kestra.plugin.core.trigger.Schedule
    cron: "0 2 * * *"

errors:
  - id: error-handler
    type: io.kestra.plugin.core.log.Log
    message: "Workflow failed: {{error.message}}"
    level: ERROR
```

## Developer Guidelines

### Local Development

1. **Reference Documentation**: Use `docs/llm_txt/` files for comprehensive plugin and pattern references
2. **Study Examples**: Review `docs/workflows/` for real-world implementation patterns
3. **Follow Conventions**: Apply naming conventions and best practices from documentation
4. **Test Thoroughly**: Validate workflows with different input scenarios

### Code Review Checklist

- [ ] Workflow follows standard anatomy from `workflow-basics.txt`
- [ ] Appropriate plugin types selected from `plugin-ecosystem.txt`
- [ ] Input validation and type safety implemented
- [ ] Error handling and retry logic included
- [ ] Security best practices applied (secrets, permissions)
- [ ] Performance considerations addressed
- [ ] Comprehensive logging and monitoring

### Integration Patterns

#### Multi-Cloud Architecture
```yaml
# Deploy across AWS, GCP, and Azure
tasks:
  - id: deploy-multi-cloud
    type: io.kestra.plugin.core.flow.Parallel
    tasks:
      - id: deploy-aws
        type: io.kestra.plugin.aws.lambda.UpdateFunctionCode
      - id: deploy-gcp
        type: io.kestra.plugin.gcp.functions.Deploy
      - id: deploy-azure
        type: io.kestra.plugin.azure.functions.Deploy
```

#### AI/ML Pipeline
```yaml
# OpenAI + Vector Database workflow
tasks:
  - id: process-data
    type: io.kestra.plugin.openai.ChatCompletion
    apiKey: "{{secret('OPENAI_API_KEY')}}"
    
  - id: store-embeddings
    type: io.kestra.plugin.weaviate.Store
    url: "{{inputs.weaviate_url}}"
```

#### Data Pipeline
```yaml
# DBT + Databricks + BigQuery
tasks:
  - id: transform-dbt
    type: io.kestra.plugin.dbt.Run
    
  - id: process-spark
    type: io.kestra.plugin.databricks.SubmitRun
    
  - id: load-bigquery
    type: io.kestra.plugin.gcp.bigquery.Load
```

## Command Line Usage

### GitHub Copilot Integration

If using GitHub Copilot, the `.github/copilot-instructions.md` file provides context-aware assistance for:
- YAML workflow generation
- Plugin selection and configuration
- Expression language usage
- Best practice implementation

### AI Assistant Prompts

Use `docs/llm_txt/ai-assistant-prompt.md` as a comprehensive prompt for AI assistants to provide expert Kestra workflow assistance, including:
- Complete plugin ecosystem knowledge
- Production-ready workflow patterns
- Security and performance best practices
- Troubleshooting and optimization guidance

## Resources and References

### Documentation Files
- **Complete Plugin Directory**: `docs/llm_txt/plugins-directory.txt`
- **Integration Patterns**: `docs/llm_txt/plugin-categories.txt`
- **Advanced Examples**: `docs/llm_txt/advanced-workflow-examples.txt`
- **Component Architecture**: `docs/component-dependencies.md`

### Sample Workflows
- **Basic ETL**: `docs/workflows/data-processing/basic-pipeline.md`
- **API Integration**: `docs/workflows/api-integration/polling-workflow.md`
- **Deployment Pipeline**: `docs/workflows/deployment/environment-deployment.md`
- **System Monitoring**: `docs/workflows/monitoring/system-monitoring.md`

### Best Practices
- **Security**: Use secrets management and proper access controls
- **Performance**: Implement parallel execution and resource optimization
- **Reliability**: Include retry logic and comprehensive error handling
- **Maintainability**: Follow naming conventions and documentation standards

## Support

For additional support with Kestra workflow development:
1. Review the comprehensive documentation in `docs/llm_txt/`
2. Study the sample workflows in `docs/workflows/`
3. Reference the plugin ecosystem documentation for specific integrations
4. Follow the best practices guidelines for production deployments

This documentation provides everything needed for AI-assisted Kestra workflow development, from basic concepts to advanced production patterns across the entire 700+ plugin ecosystem.