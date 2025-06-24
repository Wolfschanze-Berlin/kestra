# GitHub Custom Instructions and AI Prompts for Kestra

This directory contains comprehensive AI assistant instructions and prompts based on the extensive Kestra workflow documentation in the `docs/` folder.

## 📋 Available Instructions

### 🤖 GitHub Copilot Integration
- **[copilot-instructions.md](.github/copilot-instructions.md)** - Comprehensive GitHub Copilot custom instructions for Kestra workflow development
- **[COPILOT_INSTRUCTIONS](.github/COPILOT_INSTRUCTIONS)** - GitHub Copilot repository-specific instructions (standard format)

### 🧠 AI Assistant Prompts
- **[ai-assistant-prompt.md](docs/llm_txt/ai-assistant-prompt.md)** - Complete AI assistant prompt with 700+ plugin ecosystem knowledge
- **[UNIVERSAL_AI_PROMPT.md](docs/llm_txt/UNIVERSAL_AI_PROMPT.md)** - Copy-paste prompt for any AI assistant (ChatGPT, Claude, etc.)

### 📖 Development Guidelines
- **[DEVELOPMENT_INSTRUCTIONS.md](docs/DEVELOPMENT_INSTRUCTIONS.md)** - Comprehensive development guidelines and documentation usage

## 🚀 Quick Start

### For GitHub Copilot Users
1. The `.github/copilot-instructions.md` and `.github/COPILOT_INSTRUCTIONS` files provide automatic context for GitHub Copilot
2. No additional setup required - Copilot will use these instructions automatically when working in this repository

### For Other AI Assistants
1. Copy the content from `docs/llm_txt/UNIVERSAL_AI_PROMPT.md`
2. Paste it into your AI assistant (ChatGPT, Claude, Bard, etc.)
3. Add your specific Kestra workflow requirements

### For Developers
1. Review `docs/DEVELOPMENT_INSTRUCTIONS.md` for comprehensive guidelines
2. Reference the documentation in `docs/llm_txt/` for detailed plugin and pattern information
3. Use the sample workflows in `docs/workflows/` as starting points

## 📚 Documentation Structure

### Core Documentation (`docs/llm_txt/`)
- **workflow-basics.txt** - Fundamental workflow concepts and anatomy
- **plugin-ecosystem.txt** - Complete 700+ plugin ecosystem with examples
- **best-practices.txt** - Production guidelines and optimization
- **advanced-workflow-examples.txt** - Production-ready workflow patterns

### Sample Workflows (`docs/workflows/`)
- **data-processing/** - ETL pipelines with validation
- **api-integration/** - API integration with resilience patterns
- **deployment/** - Multi-environment deployment workflows
- **monitoring/** - System monitoring and alerting

## 🎯 Key Features Covered

### Comprehensive Plugin Ecosystem
- **700+ plugins** across 13 official categories
- **Cloud providers**: AWS, GCP, Azure
- **Databases**: JDBC, MongoDB, Elasticsearch, Redis
- **AI/ML**: OpenAI, LangChain4J, Weaviate
- **DevOps**: Kubernetes, Docker, Terraform
- **Data processing**: DBT, Spark, Databricks
- **And many more...**

### Production-Ready Patterns
- Error handling and retry mechanisms
- Security best practices with secrets management
- Performance optimization strategies
- Multi-cloud deployment patterns
- Real-time data processing workflows
- AI/ML pipeline orchestration

### Expression Language Support
- Pebble templating for dynamic workflows
- Input/output variable access
- Conditional logic and branching
- Date/time operations
- JSON processing and transformation

## 📝 Example Usage

### GitHub Copilot
Simply start writing a Kestra workflow YAML file in this repository, and Copilot will provide context-aware suggestions based on the custom instructions.

### ChatGPT/Claude Example
```
[Paste UNIVERSAL_AI_PROMPT.md content]

USER PROMPT: Create a Kestra workflow that processes customer feedback from a PostgreSQL database, analyzes sentiment using OpenAI, stores results in a vector database, and sends alerts for negative feedback via Slack.
```

### Development Example
```yaml
# Copilot will suggest appropriate plugin types and configuration
id: customer-feedback-analysis
namespace: customer.analytics
description: "Analyze customer feedback sentiment and alert on negative sentiment"

inputs:
  - id: feedback_table
    type: STRING
    description: "PostgreSQL table containing customer feedback"

tasks:
  # Copilot suggests: io.kestra.plugin.jdbc.postgresql.Query
  - id: extract-feedback
    type: io.kestra.plugin.jdbc.postgresql.Query
    # ... configuration suggestions
```

## 🔧 Maintenance

These instructions are automatically kept in sync with the comprehensive documentation in `docs/llm_txt/`. When the plugin ecosystem or workflow patterns are updated, the AI prompts and instructions are updated accordingly.

## 📞 Support

For questions about using these instructions or developing Kestra workflows:
1. Review the comprehensive documentation in `docs/llm_txt/`
2. Study the sample workflows in `docs/workflows/`
3. Use the AI assistant prompts for specific workflow development help

This comprehensive instruction set enables AI-assisted development of sophisticated Kestra workflows leveraging the complete 700+ plugin ecosystem for any automation, data processing, or orchestration needs.