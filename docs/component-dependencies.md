# Kestra Component Dependencies

```mermaid
graph TB
    %% External Triggers
    Scheduler[Scheduler Engine]
    Webhook[Webhook API]
    FlowTrigger[Flow Triggers]
    
    %% Core Engine Components
    Executor[Execution Engine]
    TaskRunner[Task Runner]
    FlowParser[Flow Parser]
    ExpressionEngine[Expression Engine]
    StateManager[State Manager]
    
    %% Storage and Persistence
    ExecutionStore[(Execution Store)]
    LogStore[(Log Store)]
    StateStore[(State Store)]
    FileStore[(File Store)]
    
    %% Plugin System
    CorePlugins[Core Plugins]
    FlowPlugins[Flow Control Plugins]
    UtilityPlugins[Utility Plugins]
    ExternalPlugins[External Plugins]
    
    %% Monitoring and Observability
    MetricsCollector[Metrics Collector]
    LogCollector[Log Collector]
    AlertManager[Alert Manager]
    
    %% API and UI
    WebAPI[Web API]
    WebUI[Web UI]
    CLI[CLI Client]
    
    %% Dependencies
    Scheduler --> Executor
    Webhook --> Executor
    FlowTrigger --> Executor
    
    Executor --> FlowParser
    Executor --> TaskRunner
    Executor --> StateManager
    
    FlowParser --> ExpressionEngine
    TaskRunner --> CorePlugins
    TaskRunner --> FlowPlugins
    TaskRunner --> UtilityPlugins
    TaskRunner --> ExternalPlugins
    
    TaskRunner --> ExecutionStore
    TaskRunner --> LogStore
    TaskRunner --> StateStore
    TaskRunner --> FileStore
    
    StateManager --> StateStore
    
    %% Monitoring connections
    Executor --> MetricsCollector
    TaskRunner --> LogCollector
    LogCollector --> AlertManager
    
    %% API connections
    WebAPI --> Executor
    WebAPI --> ExecutionStore
    WebAPI --> LogStore
    WebAPI --> StateStore
    
    WebUI --> WebAPI
    CLI --> WebAPI
    
    %% Plugin dependencies
    FlowPlugins --> CorePlugins
    UtilityPlugins --> CorePlugins
    ExternalPlugins --> CorePlugins
    
    classDef trigger fill:#e1f5fe
    classDef core fill:#f3e5f5
    classDef storage fill:#e8f5e8
    classDef plugin fill:#fff3e0
    classDef monitor fill:#fce4ec
    classDef interface fill:#f1f8e9
    
    class Scheduler,Webhook,FlowTrigger trigger
    class Executor,TaskRunner,FlowParser,ExpressionEngine,StateManager core
    class ExecutionStore,LogStore,StateStore,FileStore storage
    class CorePlugins,FlowPlugins,UtilityPlugins,ExternalPlugins plugin
    class MetricsCollector,LogCollector,AlertManager monitor
    class WebAPI,WebUI,CLI interface
```

## Component Descriptions

### Trigger Components
- **Scheduler Engine**: Manages cron-based and scheduled triggers
- **Webhook API**: Handles HTTP webhook triggers from external systems
- **Flow Triggers**: Manages workflow-to-workflow trigger relationships

### Core Engine Components
- **Execution Engine**: Orchestrates workflow execution and manages execution lifecycle
- **Task Runner**: Executes individual tasks and manages task state
- **Flow Parser**: Parses and validates YAML workflow definitions
- **Expression Engine**: Evaluates Pebble expressions and template rendering
- **State Manager**: Manages persistent state across workflow executions

### Storage Components
- **Execution Store**: Persists execution history, status, and metadata
- **Log Store**: Stores task and workflow execution logs
- **State Store**: Manages persistent state for workflows
- **File Store**: Handles file uploads, downloads, and working directories

### Plugin System
- **Core Plugins**: Essential flow control, debug, and utility tasks
- **Flow Control Plugins**: Sequential, parallel, conditional execution
- **Utility Plugins**: Logging, metrics, state management
- **External Plugins**: Database, API, cloud service integrations

### Monitoring and Observability
- **Metrics Collector**: Aggregates performance and execution metrics
- **Log Collector**: Centralizes log collection and processing
- **Alert Manager**: Handles notifications and alerting based on conditions

### API and Interfaces
- **Web API**: REST API for workflow management and execution
- **Web UI**: Browser-based interface for workflow development and monitoring
- **CLI Client**: Command-line interface for automation and scripting

## Data Flow

```mermaid
sequenceDiagram
    participant T as Trigger
    participant E as Execution Engine
    participant P as Flow Parser
    participant R as Task Runner
    participant S as Storage
    participant M as Monitoring
    
    T->>E: Trigger Workflow
    E->>P: Parse Flow Definition
    P->>E: Validated Flow Graph
    E->>R: Execute Task
    R->>S: Store Task Result
    R->>M: Send Metrics/Logs
    R->>E: Task Complete
    E->>S: Update Execution Status
    E->>M: Send Execution Metrics
```

## Plugin Architecture

```mermaid
graph LR
    subgraph "Plugin System"
        PluginRegistry[Plugin Registry]
        PluginLoader[Plugin Loader]
        PluginManager[Plugin Manager]
    end
    
    subgraph "Core Plugins"
        FlowControl[Flow Control]
        Debug[Debug Tasks]
        State[State Management]
        Log[Logging]
        Metrics[Metrics]
    end
    
    subgraph "External Plugins"
        Database[Database]
        HTTP[HTTP/API]
        Cloud[Cloud Services]
        FileSystem[File System]
        Notification[Notifications]
    end
    
    TaskRunner --> PluginManager
    PluginManager --> PluginRegistry
    PluginRegistry --> PluginLoader
    PluginLoader --> FlowControl
    PluginLoader --> Debug
    PluginLoader --> State
    PluginLoader --> Log
    PluginLoader --> Metrics
    PluginLoader --> Database
    PluginLoader --> HTTP
    PluginLoader --> Cloud
    PluginLoader --> FileSystem
    PluginLoader --> Notification
    
    classDef system fill:#e3f2fd
    classDef core fill:#f3e5f5
    classDef external fill:#e8f5e8
    
    class PluginRegistry,PluginLoader,PluginManager system
    class FlowControl,Debug,State,Log,Metrics core
    class Database,HTTP,Cloud,FileSystem,Notification external
```