# YAML Workflow Documentation

This directory contains comprehensive documentation and examples for creating YAML workflows in Kestra. Kestra uses YAML to define workflows in a declarative, portable, and language-agnostic manner.

## Overview

Kestra workflows are defined using YAML syntax and consist of several key components:

- **Flow Definition**: Basic flow properties like `id`, `namespace`, and `description`
- **Tasks**: The atomic units of work that make up your workflow
- **Flow Control**: Sequential, parallel, and conditional execution patterns
- **Inputs & Outputs**: Data flow between tasks and external systems
- **Variables**: Reusable values throughout the workflow
- **Triggers**: Automated workflow execution based on events or schedules
- **Error Handling**: Graceful handling of failures and retries

## Documentation Structure

- [Basic Workflows](basic-workflows.md) - Getting started with simple workflows
- [Flow Control](flow-control.md) - Sequential, parallel, and conditional execution
- [Variables and Data](variables-and-data.md) - Working with inputs, outputs, and variables
- [Error Handling](error-handling.md) - Retry policies, error tasks, and failure recovery
- [Triggers and Scheduling](triggers-scheduling.md) - Automated workflow execution
- [Best Practices](best-practices.md) - Guidelines for maintainable workflows

## Quick Start Example

Here's a simple "Hello World" workflow to get you started:

```yaml
id: hello-world
namespace: tutorial
description: A simple hello world workflow

tasks:
  - id: greet
    type: io.kestra.plugin.core.log.Log
    message: "Hello, World! The current time is {{ now() }}"
```

## Key Features

### Declarative Syntax
Workflows are defined in YAML using a simple, readable syntax that describes what should happen rather than how to implement it.

### Template Engine
Kestra includes a powerful Pebble templating engine that allows dynamic content generation using expressions like `{{ flow.id }}` and `{{ inputs.myInput }}`.

### Type Safety
Inputs and outputs are strongly typed, providing validation and better error handling.

### Extensible
Rich plugin ecosystem allows integration with databases, cloud services, APIs, and more.

## Next Steps

Start with [Basic Workflows](basic-workflows.md) to learn the fundamentals, then explore the other documentation sections based on your specific needs.